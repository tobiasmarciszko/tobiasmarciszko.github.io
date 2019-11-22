---
layout: post
title: QSettings and type safe enums
date: 2019-11-22 10:01
category: 
author: 
tags: [C++, Qt, QML]
summary: 
image: qt-logo-big.png
---

I wanted to a add a way of passing settings from QML to the C++ layer in the [Freeside Raytracer](https://github.com/tobiasmarciszko/qt_raytracer_challenge). My needs were: 

1. Store application settings between sessions (aka to disk)
2. Use/load the stored settings from disk into memory (to avoid performance hits while running)
3. Able to read and change settings from QML
4. Able to read and change settings from C++

As always, for me, when it comes to adding new features in a Qt application, I look in the [Qt documentation](https://doc.qt.io/qt-5/index.html) to see if there is something suitable for my needs. Most often there is already a platform abstraction implemented. So in this case. The [QSettings](https://doc.qt.io/qt-5/qsettings.html) class seems to be a good fit: 

_"QSettings is an abstraction around these technologies, enabling you to save and restore application settings in a portable manner. It also supports custom storage formats."_

However: 

_"If all you need is a non-persistent memory-based structure, consider using QMap<QString, QVariant> instead._"

So it seems I would need to add the in-memory storage myself to avoid reading and writing to disk to look up setting values or when settings change.

The QSettings class is also not directly available to the QML layer, so that also needs to be adressed.

Ok, let's get started! :)

The first step is to define the organization name and application name somewhere in your main.cpp or where you find it suitable. 

```cpp
// main.cpp

QCoreApplication::setOrganizationName("Tobias Marciszko");
QCoreApplication::setApplicationName("Freeside Raytracer");
```

I continue by adding a new QObject derived class called "AppSettings". This class will be the one that exposes invokable methods. That means that the exposed public methods can be called via the meta-object-system which in turn means the methods can be called from QML.

In my case I need settings that are "just" on and off. So I want a simple interface for that. 

To make things easier (or harder?) I want the settings to be strong types that I can choose from. Not just strings and values of any kind (QSettings supports keys as QString and values as QVariant).

I define my setting keys and values using the [enum class](https://isocpp.org/wiki/faq/cpp11-language-types#enum-class) available from C++11.

```cpp
// appsettings.h
enum class SettingKeys {
    FastRender
};

enum class SettingValues {
    Off,
    On
};    
```

And to make sure those are accessible to the meta-object-system the [Q_ENUM](https://doc.qt.io/qt-5/qobject.html#Q_ENUM) macros have to be set: 

```cpp
// appsettings.h
enum class SettingKeys {
    FastRender
};
Q_ENUM(SettingKeys)

enum class SettingValues {
    Off,
    On
};    
Q_ENUM(SettingValues)
```
Now we have two strongly typed enums which are accessible from both C++ and QML in one go!

As with all classes and types that is exposed to QML, they need to be registered as well:

```cpp
// main.cpp

// 1. Type registration of the class, to make sure the enums are properly exposed
qmlRegisterType<AppSettings>("myextension", 1, 0, "AppSettings");

// 2. Property registration, to be able to call the invokable methods on this instance
AppSettings settings;
engine.rootContext()->setContextProperty("settings", &settings);
```

Having the AppSettings type exposed, the enums can now be accessed from the QML code such as this. Note, we haven't added the "isEnabled" and "setEnabled" method just yet :)

```qml
// raytracer.qml
Component.onCompleted: {
    checked = settings.isEnabled(AppSettings.FastRender)
}

onClicked: {
    settings.setEnabled(AppSettings.FastRender, 
                        checked ? AppSettings.On : AppSettings.Off)
}
```

The invokable method "setEnabled" is defined as: 

```cpp
// appsettings.h

Q_INVOKABLE static void setEnabled(const SettingKeys& key, const SettingValues& value) {
    // Sets the value to both in-memory structure and to disk
    const auto k = AppSettings::enumToString(key);
    const auto v = AppSettings::enumToString(value);

    QSettings s;
    s.setValue(k, v);
    m_cache[key] = value;
}
```

It's declared with the the [Q_INVOKABLE](https://doc.qt.io/qt-5/qobject.html#Q_INVOKABLE) macro to make it visible to QML. The parameters are strongly typed, thanks to the enum class!

Since the API for QSettings expect a QString as key and a QVariant as value I cannot just pass in the enums to it. Even if I could I would still want the _name_ of the enum passed in, nost just the value of it. I'd prefer a setting being called "FastRender" in the setting file on disk rather than just "0". 

To do so, we can use the [QMetaEnum](https://doc.qt.io/qt-5/qmetaenum.html) class to provide the name for us in a format that we need (a QString).

To map a Q_ENUM value to it's string representation we can define a conversion method for it: 

```cpp
// appsettings.h

template <class EnumClass>
static QString enumToString(const EnumClass& key) {
    const auto metaEnum = QMetaEnum::fromType<EnumClass>();
    return metaEnum.valueToKey(static_cast<int>(key));
}
```

This method will take any Q_ENUM class key parameter as argument and return the QString representation of it. Note that the "static_cast<int>" on the key is perfectly legal here, even though the enum is strongly typed with "enum class".

In essence, by calling the "setEnabled" with a SettingKey and a SettingValue will produce a line in the config file on disk such as: 


```
// Freeside Raytracer.conf (on Linux)

[General]
FastRender=On
```

The full code for the AppSettings class is [here](https://github.com/tobiasmarciszko/qt_raytracer_challenge/blob/settings/raytracer/main/appsettings.h) for all the details.

Soo... Did I reach my goals? 

#### ✓ Store application settings between sessions (aka to disk)
QSettings takes care of it, regardles of platform.

#### ✓ Use/load the stored settings from disk into memory (to avoid performance hits while running)
On initialisation the disk settings are read into a memory structure for quick access. On exit memory contents are written to disk.

#### ✓ Able to read and change settings from QML
Thanks to Q_ENUM and Q_INVOKABLE we can expose the methods and enums we want from our C++ class to the [meta-object-system](https://doc.qt.io/qt-5/metaobjects.html)!

#### ✓ Able to read and change settings from C++
The AppSettings class exposes the methods needed from its public interface.

That covers it for me, for now. Next step is to add more settings and the UI components in QML to fully make use of the settings backend we just created.

That's all folks! :)
