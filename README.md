Hypodermic
==========

Hypodermic is an IoC container for C++. It provides dependency injection to your existing design.

[![MIT license](http://img.shields.io/badge/license-MIT-brightgreen.svg)](http://opensource.org/licenses/MIT)
[![Build status](https://img.shields.io/appveyor/ci/ybainier/Hypodermic/master.svg?label=windows)](https://ci.appveyor.com/project/ybainier/hypodermic)
[![Build Status](https://img.shields.io/travis/ybainier/Hypodermic/master.svg?label=linux/osx)](https://travis-ci.org/ybainier/Hypodermic)

## Ackowledgment

Hypodermic was started with the will to mimic the famous .NET Autofac http://autofac.org/. Although its design evolved, both its behavior and its expressiveness tend to remain the same.
Constructor deduction has been made possible thanks to the work of [Krzysztof Jusiak](http://krzysztof.jusiak.net/) in http://boost-experimental.github.io/di/.

## Register components

Configure the container by registering types, instances, and instance factories.
```cpp
ContainerBuilder builder;

builder.registerInstance(std::make_shared< BusConfiguration >());

builder.registerInstanceFactory([](Container&)
{
    return std::make_shared< ThreadPool >(std::thread::hardware_concurrency());
});

builder.registerType< MessageDispatcher >()
       .as< IMessageDispatcher >();

builder.registerType< MessageSerializer >();

builder.registerType< Transport >()
       .with< IMessageSerializer, MessageSerializer >()
       .as< ITransport >();
       
builder.registerType< Bus >()
       .as< IBus >()
       .singleInstance();
       
auto container = builder.build();
```

## Express dependencies

Hypodermic automatically inject your constructor dependencies.
```cpp
class Service
{
public:
    explicit Service(const std::shared_ptr< INotificationSender >& notificationSender)
        : m_notificationSender(notificationSender)
    { }
  
private:
    std::shared_ptr< INotificationSender > m_notificationSender;
};
```

## Resolve types

![Sample class hierarchy](../master/resources/home_page_simple_diagram.png?raw=true "Sample class hierarchy")

In your application, you may write code like this:
```cpp
ContainerBuilder builder;

builder.registerType< SmsNotificationSender >().as< INotificationSender >();

auto container = builder.build();

auto service = container->resolve< Service >();
service->start();
```
Now, in a unit test:
```cpp
// Arrange
auto mock = std::make_shared< NotificationSenderMock >();
Service service(mock);

// Act & Assert
EXPECT_CALL(*mock, send(_)).Times(1);

BOOST_CHECK_NO_THROW(service.start());
```

## Wiring the internal logging mechanism

You may want to trace Hypodermic's activity by wiring your own logging mechanism to it. You have to implement `ILoggerSink` and configure the singleton of `Logger` like so:
```cpp
// You will intercept every messages but at no 0 cost. Warn might be a little less aggressive.
Logger::configureLogLevel(LogLevels::Debug);

Logger::configureSink(std::make_shared< MyVeryOwnLog4cxxLoggerSink >());
```


## Features

* Header only
* Developer friendly compilation errors
* Easy to use out of the box
* Configurable instance lifetime
* On the fly registrations
* Constructor discovery
* Merge-able registrations
* Dependencies can be particular for a specific type
* Injectable vector of types
* Injectable factories of types
* Injectable container
* Used in production environments for over 4 years

## Comparative benchmarks

The source is available in [Hypodermic.Tests/PerformanceTests.cpp](https://github.com/ybainier/Hypodermic/blob/master/Hypodermic.Tests/PerformanceTests.cpp)

### Default constructible type

|              | std::make_shared | resolve not registered transient | resolve registered transient | resolve registered single instance | resolve registered instance |
|--------------|-----------------:|---------------------------------:|-----------------------------:|-----------------------------------:|----------------------------:|
| Overall (ms) |               10 |                               71 |                           70 |                                 54 |                          55 |
| Average (ns) |               72 |                              676 |                          672 |                                512 |                         518 |
| Min     (ns) |                0 |                              570 |                          570 |                                285 |                         285 |
| Max     (µs) |               43 |                              149 |                          267 |                                 87 |                         268 |
| 99.999% (µs) |               40 |                               67 |                           67 |                                 70 |                         115 |
| 99.99%  (µs) |               22 |                               41 |                           30 |                                 24 |                          24 |
| 99.9%   (µs) |                1 |                                9 |                            7 |                                  2 |                           4 |
| 99%     (ns) |              286 |                              856 |                          856 |                                571 |                         571 |
| 95%     (ns) |              285 |                              856 |                          856 |                                571 |                         571 |
| 50%     (ns) |                0 |                              570 |                          570 |                                570 |                         570 |

### Type with default constructible dependency

|              | std::make_shared | resolve not registered transient | resolve registered transient | resolve with registered single instance dependency | resolve with registered instance dependency |
|--------------|-----------------:|---------------------------------:|-----------------------------:|---------------------------------------------------:|--------------------------------------------:|
| Overall (ms) |               18 |                              192 |                          190 |                                                177 |                                         170 |
| Average (µs) |              0.1 |                                1 |                            1 |                                                  1 |                                           1 |
| Min     (µs) |                0 |                                1 |                            1 |                                                  1 |                                           1 |
| Max     (µs) |              150 |                              314 |                          513 |                                                160 |                                          92 |
| 99.999% (µs) |               86 |                              181 |                          139 |                                                104 |                                          85 |
| 99.99%  (µs) |               27 |                               75 |                           53 |                                                 47 |                                          55 |
| 99.9%   (µs) |                8 |                               23 |                           23 |                                                 21 |                                          21 |
| 99%     (µs) |              0.3 |                                2 |                            2 |                                                  2 |                                           2 |
| 95%     (µs) |              0.3 |                                1 |                            1 |                                                  1 |                                           1 |
| 50%     (µs) |                0 |                                1 |                            1 |                                                  1 |                                           1 |
