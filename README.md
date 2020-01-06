# QtSignalsSlotsEx
Qt Signals & Slots:
Signals & Slots:(https://en.wikipedia.org/wiki/Signals_and_slots)
-----------------

Signals and slots is a language construct introduced in Qt for communication between objects[1] which makes it easy to implement the observer pattern while avoiding boilerplate code. The concept is that GUI widgets can send signals containing event information which can be received by other widgets / controls using special functions known as slots. This is similar to C/C++ function pointers, but signal/slot system ensures the type-correctness of callback arguments.

The signal/slot system fits well with the way graphical user interfaces are designed. Similarly, the signal/slot system can be used for other non-GUI usages, for example asynchronous I/O (including sockets, pipes, serial devices, etc.) event notification or to associate timeout events with appropriate object instances and methods or functions. It is easy to use and no registration/deregistration/invocation code need to be written, because Qt's metaobject compiler (MOC) automatically generates the needed infrastructure.

A commonly used metaphor is a spreadsheet. A spreadsheet has cells that observe the source cell(s). When the source cell is changed, the dependent cells are updated from the event.

Signals & Slots:(https://doc.qt.io/qt-5/signalsandslots.html)
-----------------
Signals and slots are used for communication between objects. The signals and slots mechanism is a central feature of Qt and probably the part that differs most from the features provided by other frameworks. Signals and slots are made possible by Qt's meta-object system.

Other toolkits achieve this kind of communication using callbacks. A callback is a pointer to a function, so if you want a processing function to notify you about some event you pass a pointer to another function (the callback) to the processing function. The processing function then calls the callback when appropriate. While successful frameworks using this method do exist, callbacks can be unintuitive and may suffer from problems in ensuring the type-correctness of callback arguments.

In Qt, we have an alternative to the callback technique: We use signals and slots. A signal is emitted when a particular event occurs. Qt's widgets have many predefined signals, but we can always subclass widgets to add our own signals to them. A slot is a function that is called in response to a particular signal. Qt's widgets have many pre-defined slots, but it is common practice to subclass widgets and add your own slots so that you can handle the signals that you are interested in.

The signals and slots mechanism is type safe: The signature of a signal must match the signature of the receiving slot. (In fact a slot may have a shorter signature than the signal it receives because it can ignore extra arguments.) Since the signatures are compatible, the compiler can help us detect type mismatches when using the function pointer-based syntax. The string-based SIGNAL and SLOT syntax will detect type mismatches at runtime. Signals and slots are loosely coupled: A class which emits a signal neither knows nor cares which slots receive the signal. Qt's signals and slots mechanism ensures that if you connect a signal to a slot, the slot will be called with the signal's parameters at the right time. Signals and slots can take any number of arguments of any type. They are completely type safe.

All classes that inherit from QObject or one of its subclasses (e.g., QWidget) can contain signals and slots. Signals are emitted by objects when they change their state in a way that may be interesting to other objects. This is all the object does to communicate. It does not know or care whether anything is receiving the signals it emits. This is true information encapsulation, and ensures that the object can be used as a software component.

Slots can be used for receiving signals, but they are also normal member functions. Just as an object does not know if anything receives its signals, a slot does not know if it has any signals connected to it. This ensures that truly independent components can be created with Qt.

You can connect as many signals as you want to a single slot, and a signal can be connected to as many slots as you need. It is even possible to connect a signal directly to another signal. (This will emit the second signal immediately whenever the first is emitted.)

Together, signals and slots make up a powerful component programming mechanism.

Signals:
----------
Signals are emitted by an object when its internal state has changed in some way that might be interesting to the object's client or owner. Signals are public access functions and can be emitted from anywhere, but we recommend to only emit them from the class that defines the signal and its subclasses.

When a signal is emitted, the slots connected to it are usually executed immediately, just like a normal function call. When this happens, the signals and slots mechanism is totally independent of any GUI event loop. Execution of the code following the emit statement will occur once all slots have returned. The situation is slightly different when using queued connections; in such a case, the code following the emit keyword will continue immediately, and the slots will be executed later.

If several slots are connected to one signal, the slots will be executed one after the other, in the order they have been connected, when the signal is emitted.

Signals are automatically generated by the moc and must not be implemented in the .cpp file. They can never have return types (i.e. use void).

A note about arguments: Our experience shows that signals and slots are more reusable if they do not use special types. If QScrollBar::valueChanged() were to use a special type such as the hypothetical QScrollBar::Range, it could only be connected to slots designed specifically for QScrollBar. Connecting different input widgets together would be impossible.

Slots:
---------
A slot is called when a signal connected to it is emitted. Slots are normal C++ functions and can be called normally; their only special feature is that signals can be connected to them.

Since slots are normal member functions, they follow the normal C++ rules when called directly. However, as slots, they can be invoked by any component, regardless of its access level, via a signal-slot connection. This means that a signal emitted from an instance of an arbitrary class can cause a private slot to be invoked in an instance of an unrelated class.

You can also define slots to be virtual, which we have found quite useful in practice.

Compared to callbacks, signals and slots are slightly slower because of the increased flexibility they provide, although the difference for real applications is insignificant. In general, emitting a signal that is connected to some slots, is approximately ten times slower than calling the receivers directly, with non-virtual function calls. This is the overhead required to locate the connection object, to safely iterate over all connections (i.e. checking that subsequent receivers have not been destroyed during the emission), and to marshall any parameters in a generic fashion. While ten non-virtual function calls may sound like a lot, it's much less overhead than any new or delete operation, for example. As soon as you perform a string, vector or list operation that behind the scene requires new or delete, the signals and slots overhead is only responsible for a very small proportion of the complete function call costs. The same is true whenever you do a system call in a slot; or indirectly call more than ten functions. The simplicity and flexibility of the signals and slots mechanism is well worth the overhead, which your users won't even notice.

Note that other libraries that define variables called signals or slots may cause compiler warnings and errors when compiled alongside a Qt-based application. To solve this problem, #undef the offending preprocessor symbol.

Q_OBJECT:
-----------
The Q_OBJECT macro must appear in the private section of a class definition that declares its own signals and slots or that uses other services provided by Qt's meta-object system.

For example:

#include <QObject>

class Counter : public QObject
{
    Q_OBJECT

public:
    Counter() { m_value = 0; }

    int value() const { return m_value; }

public slots:
    void setValue(int value);

signals:
    void valueChanged(int newValue);

private:
    int m_value;
};
Note: This macro requires the class to be a subclass of QObject. Use Q_GADGET instead of Q_OBJECT to enable the meta object system's support for enums in a class that is not a QObject subclass.

Q_GADGET:
------------
The Q_GADGET macro is a lighter version of the Q_OBJECT macro for classes that do not inherit from QObject but still want to use some of the reflection capabilities offered by QMetaObject. Just like the Q_OBJECT macro, it must appear in the private section of a class definition.

Q_GADGETs can have Q_ENUM, Q_PROPERTY and Q_INVOKABLE, but they cannot have signals or slots.

Q_GADGET makes a class member, staticMetaObject, available. staticMetaObject is of type QMetaObject and provides access to the enums declared with Q_ENUMS.

Automatic Connections:
----------------------

The signals and slots connections defined for compile time or run time forms can either be set up manually or automatically, using QMetaObject's ability to make connections between signals and suitably-named slots.

Generally, in a QDialog, if we want to process the information entered by the user before accepting it, we need to connect the clicked() signal from the OK button to a custom slot in our dialog. We will first show an example of the dialog in which the slot is connected by hand then compare it with a dialog that uses automatic connection.

Widgets and Dialogs with Auto-Connect:
--------------------------------------

Although it is easy to implement a custom slot in the dialog and connect it in the constructor, we could instead use QMetaObject's auto-connection facilities to connect the OK button's clicked() signal to a slot in our subclass. uic automatically generates code in the dialog's setupUi() function to do this, so we only need to declare and implement a slot with a name that follows a standard convention:

void on_<object name>_<signal name>(<signal parameters>);

Using Qt with 3rd Party Signals and Slots:
---------------------------------------------
It is possible to use Qt with a 3rd party signal/slot mechanism. You can even use both mechanisms in the same project. Just add the following line to your qmake project (.pro) file.

CONFIG += no_keywords
It tells Qt not to define the moc keywords signals, slots, and emit, because these names will be used by a 3rd party library, e.g. Boost. Then to continue using Qt signals and slots with the no_keywords flag, simply replace all uses of the Qt moc keywords in your sources with the corresponding Qt macros Q_SIGNALS (or Q_SIGNAL), Q_SLOTS (or Q_SLOT), and Q_EMIT.

QObject::connect
----------------
[static]QMetaObject::Connection QObject::connect(const QObject *sender, const char *signal, const QObject *receiver, const char *method, Qt::ConnectionType type = Qt::AutoConnection)
Creates a connection of the given type from the signal in the sender object to the method in the receiver object. Returns a handle to the connection that can be used to disconnect it later.

You must use the SIGNAL() and SLOT() macros when specifying the signal and the method, for example:

QLabel *label = new QLabel;
QScrollBar *scrollBar = new QScrollBar;
QObject::connect(scrollBar, SIGNAL(valueChanged(int)),
                 label,  SLOT(setNum(int)));
This example ensures that the label always displays the current scroll bar value. Note that the signal and slots parameters must not contain any variable names, only the type. E.g. the following would not work and return false:

// WRONG
QObject::connect(scrollBar, SIGNAL(valueChanged(int value)),
                 label, SLOT(setNum(int value)));
A signal can also be connected to another signal:

class MyWidget : public QWidget
{
    Q_OBJECT

public:
    MyWidget();

signals:
    void buttonClicked();

private:
    QPushButton *myButton;
};

MyWidget::MyWidget()
{
    myButton = new QPushButton(this);
    connect(myButton, SIGNAL(clicked()),
            this, SIGNAL(buttonClicked()));
}
In this example, the MyWidget constructor relays a signal from a private member variable, and makes it available under a name that relates to MyWidget.

A signal can be connected to many slots and signals. Many signals can be connected to one slot.

If a signal is connected to several slots, the slots are activated in the same order in which the connections were made, when the signal is emitted.

The function returns a QMetaObject::Connection that represents a handle to a connection if it successfully connects the signal to the slot. The connection handle will be invalid if it cannot create the connection, for example, if QObject is unable to verify the existence of either signal or method, or if their signatures aren't compatible. You can check if the handle is valid by casting it to a bool.

By default, a signal is emitted for every connection you make; two signals are emitted for duplicate connections. You can break all of these connections with a single disconnect() call. If you pass the Qt::UniqueConnection type, the connection will only be made if it is not a duplicate. If there is already a duplicate (exact same signal to the exact same slot on the same objects), the connection will fail and connect will return an invalid QMetaObject::Connection.

Note: Qt::UniqueConnections do not work for lambdas, non-member functions and functors; they only apply to connecting to member functions.

The optional type parameter describes the type of connection to establish. In particular, it determines whether a particular signal is delivered to a slot immediately or queued for delivery at a later time. If the signal is queued, the parameters must be of types that are known to Qt's meta-object system, because Qt needs to copy the arguments to store them in an event behind the scenes. If you try to use a queued connection and get the error message

QObject::connect: Cannot queue arguments of type 'MyType'
(Make sure 'MyType' is registered using qRegisterMetaType().)
call qRegisterMetaType() to register the data type before you establish the connection.

Note: This function is thread-safe.

See also disconnect(), sender(), qRegisterMetaType(), Q_DECLARE_METATYPE(), and Differences between String-Based and Functor-Based Connections.


[static]QMetaObject::Connection QObject::connect(const QObject *sender, const QMetaMethod &signal, const QObject *receiver, const QMetaMethod &method, Qt::ConnectionType type = Qt::AutoConnection)
--------------------------------------------------------------------------------------

Creates a connection of the given type from the signal in the sender object to the method in the receiver object. Returns a handle to the connection that can be used to disconnect it later.

The Connection handle will be invalid if it cannot create the connection, for example, the parameters were invalid. You can check if the QMetaObject::Connection is valid by casting it to a bool.

This function works in the same way as connect(const QObject *sender, const char *signal, const QObject *receiver, const char *method, Qt::ConnectionType type) but it uses QMetaMethod to specify signal and method.

This function was introduced in Qt 4.8.

See also connect(const QObject *sender, const char *signal, const QObject *receiver, const char *method, Qt::ConnectionType type).

QMetaObject::Connection QObject::connect(const QObject *sender, const char *signal, const char *method, Qt::ConnectionType type = Qt::AutoConnection) const
------------------------------------------------------------------------------------------------------
This function overloads connect().

Connects signal from the sender object to this object's method.

Equivalent to connect(sender, signal, this, method, type).

Every connection you make emits a signal, so duplicate connections emit two signals. You can break a connection using disconnect().

Note: This function is thread-safe.


[static]QMetaObject::Connection QObject::connect(const QObject *sender, PointerToMemberFunction signal, const QObject *receiver, PointerToMemberFunction method, Qt::ConnectionType type = Qt::AutoConnection)
------------------------------------------------------------------------------------------------------
This function overloads connect().

Creates a connection of the given type from the signal in the sender object to the method in the receiver object. Returns a handle to the connection that can be used to disconnect it later.

The signal must be a function declared as a signal in the header. The slot function can be any member function that can be connected to the signal. A slot can be connected to a given signal if the signal has at least as many arguments as the slot, and there is an implicit conversion between the types of the corresponding arguments in the signal and the slot.

Example:

QLabel *label = new QLabel;
QLineEdit *lineEdit = new QLineEdit;
QObject::connect(lineEdit, &QLineEdit::textChanged,
                 label,  &QLabel::setText);
				 
This example ensures that the label always displays the current line edit text.

A signal can be connected to many slots and signals. Many signals can be connected to one slot.

If a signal is connected to several slots, the slots are activated in the same order as the order the connection was made, when the signal is emitted

The function returns an handle to a connection if it successfully connects the signal to the slot. The Connection handle will be invalid if it cannot create the connection, for example, if QObject is unable to verify the existence of signal (if it was not declared as a signal) You can check if the QMetaObject::Connection is valid by casting it to a bool.

By default, a signal is emitted for every connection you make; two signals are emitted for duplicate connections. You can break all of these connections with a single disconnect() call. If you pass the Qt::UniqueConnection type, the connection will only be made if it is not a duplicate. If there is already a duplicate (exact same signal to the exact same slot on the same objects), the connection will fail and connect will return an invalid QMetaObject::Connection.

The optional type parameter describes the type of connection to establish. In particular, it determines whether a particular signal is delivered to a slot immediately or queued for delivery at a later time. If the signal is queued, the parameters must be of types that are known to Qt's meta-object system, because Qt needs to copy the arguments to store them in an event behind the scenes. If you try to use a queued connection and get the error message

QObject::connect: Cannot queue arguments of type 'MyType'
(Make sure 'MyType' is registered using qRegisterMetaType().)
make sure to declare the argument type with Q_DECLARE_METATYPE

Overloaded functions can be resolved with help of qOverload.

Note: This function is thread-safe.

See also Differences between String-Based and Functor-Based Connections.

[static]QMetaObject::Connection QObject::connect(const QObject *sender, PointerToMemberFunction signal, Functor functor)
This function overloads connect().

Creates a connection from signal in sender object to functor, and returns a handle to the connection

The signal must be a function declared as a signal in the header. The slot function can be any function or functor that can be connected to the signal. A function can be connected to a given signal if the signal has at least as many argument as the slot. A functor can be connected to a signal if they have exactly the same number of arguments. There must exist implicit conversion between the types of the corresponding arguments in the signal and the slot.

Example:

void someFunction();
QPushButton *button = new QPushButton;
QObject::connect(button, &QPushButton::clicked, someFunction);
Lambda expressions can also be used:

QByteArray page = ...;
QTcpSocket *socket = new QTcpSocket;
socket->connectToHost("qt-project.org", 80);
QObject::connect(socket, &QTcpSocket::connected, [=] () {
        socket->write("GET " + page + "\r\n");
    });
The connection will automatically disconnect if the sender is destroyed. However, you should take care that any objects used within the functor are still alive when the signal is emitted.

Overloaded functions can be resolved with help of qOverload.

Note: This function is thread-safe.

[static]QMetaObject::Connection QObject::connect(const QObject *sender, PointerToMemberFunction signal, const QObject *context, Functor functor, Qt::ConnectionType type = Qt::AutoConnection)
This function overloads connect().

Creates a connection of a given type from signal in sender object to functor to be placed in a specific event loop of context, and returns a handle to the connection.

Note: Qt::UniqueConnections do not work for lambdas, non-member functions and functors; they only apply to connecting to member functions.

The signal must be a function declared as a signal in the header. The slot function can be any function or functor that can be connected to the signal. A function can be connected to a given signal if the signal has at least as many argument as the slot. A functor can be connected to a signal if they have exactly the same number of arguments. There must exist implicit conversion between the types of the corresponding arguments in the signal and the slot.

Example:

void someFunction();
QPushButton *button = new QPushButton;
QObject::connect(button, &QPushButton::clicked, this, someFunction, Qt::QueuedConnection);
Lambda expressions can also be used:

QByteArray page = ...;
QTcpSocket *socket = new QTcpSocket;
socket->connectToHost("qt-project.org", 80);
QObject::connect(socket, &QTcpSocket::connected, this, [=] () {
        socket->write("GET " + page + "\r\n");
    }, Qt::AutoConnection);
The connection will automatically disconnect if the sender or the context is destroyed. However, you should take care that any objects used within the functor are still alive when the signal is emitted.

Overloaded functions can be resolved with help of qOverload.

Note: This function is thread-safe.

int qRegisterMetaType(const char *typeName):
--------------------------------------
Registers the type name typeName for the type T. Returns the internal ID used by QMetaType. Any class or struct that has a public default constructor, a public copy constructor and a public destructor can be registered.

This function requires that T is a fully defined type at the point where the function is called. For pointer types, it also requires that the pointed to type is fully defined. Use Q_DECLARE_OPAQUE_POINTER() to be able to register pointers to forward declared types.

After a type has been registered, you can create and destroy objects of that type dynamically at run-time.

This example registers the class MyClass:

qRegisterMetaType<MyClass>("MyClass");
This function is useful to register typedefs so they can be used by QMetaProperty, or in QueuedConnections

typedef QString CustomString;
qRegisterMetaType<CustomString>("CustomString");
Warning: This function is useful only for registering an alias (typedef) for every other use case Q_DECLARE_METATYPE and qMetaTypeId() should be used instead.

void QObject::connectNotify(const QMetaMethod &signal):
------------------------------------------------------
This virtual function is called when something has been connected to signal in this object.

If you want to compare signal with a specific signal, you can use QMetaMethod::fromSignal() as follows:

if (signal == QMetaMethod::fromSignal(&MyObject::valueChanged)) {
    // signal is valueChanged
}
Warning: This function violates the object-oriented principle of modularity. However, it might be useful when you need to perform expensive initialization only if something is connected to a signal.

Warning: This function is called from the thread which performs the connection, which may be a different thread from the thread in which this object lives.

[virtual protected]void QObject::disconnectNotify(const QMetaMethod &signal)
---------------------------------------------------------------------------
This virtual function is called when something has been disconnected from signal in this object.

See connectNotify() for an example of how to compare signal with a specific signal.

If all signals were disconnected from this object (e.g., the signal argument to disconnect() was 0), disconnectNotify() is only called once, and the signal will be an invalid QMetaMethod (QMetaMethod::isValid() returns false).

Warning: This function violates the object-oriented principle of modularity. However, it might be useful for optimizing access to expensive resources.

Warning: This function is called from the thread which performs the disconnection, which may be a different thread from the thread in which this object lives. This function may also be called with a QObject internal mutex locked. It is therefore not allowed to re-enter any of any QObject functions from your reimplementation and if you lock a mutex in your reimplementation, make sure that you don't call QObject functions with that mutex held in other places or it will result in a deadlock.

This function was introduced in Qt 5.0.

[static]bool QObject::disconnect(const QObject *sender, const char *signal, const QObject *receiver, const char *method)
------------------------------------------------------------------------------------------------------------------
Disconnects signal in object sender from method in object receiver. Returns true if the connection is successfully broken; otherwise returns false.

A signal-slot connection is removed when either of the objects involved are destroyed.

disconnect() is typically used in three ways, as the following examples demonstrate.

Disconnect everything connected to an object's signals:
disconnect(myObject, 0, 0, 0);
equivalent to the non-static overloaded function

myObject->disconnect();
Disconnect everything connected to a specific signal:
disconnect(myObject, SIGNAL(mySignal()), 0, 0);
equivalent to the non-static overloaded function

myObject->disconnect(SIGNAL(mySignal()));
Disconnect a specific receiver:
disconnect(myObject, 0, myReceiver, 0);
equivalent to the non-static overloaded function

myObject->disconnect(myReceiver);
0 may be used as a wildcard, meaning "any signal", "any receiving object", or "any slot in the receiving object", respectively.

The sender may never be nullptr. (You cannot disconnect signals from more than one object in a single call.)

If signal is 0, it disconnects receiver and method from any signal. If not, only the specified signal is disconnected.

If receiver is 0, it disconnects anything connected to signal. If not, slots in objects other than receiver are not disconnected.

If method is 0, it disconnects anything that is connected to receiver. If not, only slots named method will be disconnected, and all other slots are left alone. The method must be nullptr if receiver is left out, so you cannot disconnect a specifically-named slot on all objects.

Note: This function is thread-safe.

bool QObject::disconnect(const QObject *sender, const QMetaMethod &signal, const QObject *receiver, const QMetaMethod &method)
-----------------------------------------------------------------------------------------------
Disconnects signal in object sender from method in object receiver. Returns true if the connection is successfully broken; otherwise returns false.

This function provides the same possibilities like disconnect(const QObject *sender, const char *signal, const QObject *receiver, const char *method) but uses QMetaMethod to represent the signal and the method to be disconnected.

Additionally this function returnsfalse and no signals and slots disconnected if:

signal is not a member of sender class or one of its parent classes.
method is not a member of receiver class or one of its parent classes.
signal instance represents not a signal.
QMetaMethod() may be used as wildcard in the meaning "any signal" or "any slot in receiving object". In the same way 0 can be used for receiver in the meaning "any receiving object". In this case method should also be QMetaMethod(). sender parameter should be never 0.

Differences between String-Based and Functor-Based Connections:
------------------------------------------------------------------
From Qt 5.0 onwards, Qt offers two different ways to write signal-slot connections in C++: The string-based connection syntax and the functor-based connection syntax. There are pros and cons to both syntaxes. The table below summarizes their differences.

																							String-based	Functor-based
Type checking is done at...																		Run-time	Compile-time
Can perform implicit type conversions																     		Y
Can connect signals to lambda expressions																		Y
Can connect signals to slots which have more arguments than the signal (using default parameters)	Y	
Can connect C++ functions to QML functions															Y	

[protected]int QObject::receivers(const char *signal) const
---------------------------------------------------------------
Returns the number of receivers connected to the signal.

Since both slots and signals can be used as receivers for signals, and the same connections can be made many times, the number of receivers is the same as the number of connections made from this signal.

When calling this function, you can use the SIGNAL() macro to pass a specific signal:

if (receivers(SIGNAL(valueChanged(QByteArray))) > 0) {
    QByteArray data;
    get_the_value(&data);       // expensive operation
    emit valueChanged(data);
}
Warning: This function violates the object-oriented principle of modularity. However, it might be useful when you need to perform expensive initialization only if something is connected to a signal.


[protected]QObject *QObject::sender() const
-----------------------------------------------
Returns a pointer to the object that sent the signal, if called in a slot activated by a signal; otherwise it returns nullptr. The pointer is valid only during the execution of the slot that calls this function from this object's thread context.

The pointer returned by this function becomes invalid if the sender is destroyed, or if the slot is disconnected from the sender's signal.

Warning: This function violates the object-oriented principle of modularity. However, getting access to the sender might be useful when many signals are connected to a single slot.

Warning: As mentioned above, the return value of this function is not valid when the slot is called via a Qt::DirectConnection from a thread different from this object's thread. Do not use this function in this type of scenario.

How Qt Signals and Slots Work:
--------------------------------
https://woboq.com/blog/how-qt-signals-slots-work.html
https://woboq.com/blog/how-qt-signals-slots-work-part2-qt5.html
https://woboq.com/blog/new-signals-slots-syntax-in-qt5.html


