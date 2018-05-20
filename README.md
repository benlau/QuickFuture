Quick Future
============
[![Build Status](https://travis-ci.org/benlau/quickfuture.svg?branch=master)](https://travis-ci.org/benlau/quickfuture)

QuickFuture is a QML wrapper of QFuture. It allows user to access and listen from a QFuture object generated by a QObject class. So that QML could respond to the result of an asynchronous task.

**Example**

```
import QuickFuture 1.0

...

var future = FileActor.read(“tmp.txt”);
// FileActor is a C++ class registered as context property
// QFuture<QString> FileActor::read(const QString&file);
// It is not a part of the library

Future.onFinished(future, function(value) {
  // do something when it is finished.
});

Future.promise(future).then(function(value) {
  // Future.promise creates a promise object by using Quick Promise.
  // It is useful when you have multiple asynchronuous tasks pending.
});

var promise = Future.promise(future);

promise.reject(timer.trigger);

promise.then(function(value) {
  // It is called if the future has finished before timer timeout
},function() {
  // Once it timeout, run this function instead of previous one
});

...

```

Installation
============

    qpm install quick.future.pri


Custom Type Registration
========================

By default, QFuture<T> is not a standard type recognized by QML.
It must be registered as a QMetaType per template type in order to get rid the error message.

The same rule applies in Quick Future too.
Common types are pre-registered already.
For your own custom type, you can register it by:

```c++
#include <QuickFuture>
Q_DECLARE_METATYPE(CustomType)
Q_DECLARE_METATYPE(QFuture<CustomType>)

...

int main(int argc, char *argv[])
{

//...
    QuickFuture::registerType<CustomType>([](CustomType value) -> QVariant {
        // Optional converter function.
        QVariantMap res;
        res["field"] = value.field;
        // ....
        return res;
   });
//...

}
```

Pre-registered data type list: bool, int, qreal, QString, QByteArray, QVariantMap, QSize, void.

Custom Converter Function
-------------------------

When you are registering a custom type, you may assign a custom converter function for making a QML friendly data structure (e.g QVariantMap) from the custom type. The value could be obtained by using `Future.result()`

Example

```c++
class Actor : public QObject {
Q_OBJECT
public:
  class Reply {
    public:
      int code;
      QString message;
  };
  QFuture<Reply> read(QString source);
}

static void init() {
    QuickFuture::registerType<Actor::Reply>([](Actor::Reply reply) -> QVariant {
        // Optional converter function.
        QVariantMap map;
        map["code"] = reply.code;
        map["message"] = reply.message;
        return map;
    });
}

Q_COREAPP_STARTUP_FUNCTION(init)
```

```QML

var future = Actor.read(source);
....
console.log(Future.result(future)); // Print { code: 0, message: ""} if the reply is empty
```


API
===

(More API will be added upon request)

**Future.isFinished(future)**

Returns true if the asynchronous computation represented by this future has finished; otherwise returns false.

**Future.isRunning(future)**

**Future.isCanceled(future)**

**Future.onFinished(future, callback)**

The callback will be invoked when the watched future finishes.

**Future.onCanceled(future, callback)**

**Future.onProgressValueChanged(future, callback)**

**Future.promise(future)**

Create a promise object which will be resolved when the future has finished or will be rejected when the future is canceled. It must have QuickPromise installed and setup properly before using this function.

**Future.result(future)**

Object the result of a Future object.

**Future.sync(future, propertyAtFuture, target, propertyAtTarget)**

Synchronize a property in future object to target object.

Example:
```

QtObject {
    id: target1
    property var isRunning
    property var isFinished
}

// Future.sync(future,"isRunning", target1, "isRunning");
```

Supported properties: "isRunning", "isCanceled", "isFinished"

