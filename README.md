
---------------------
README file for PyICU
---------------------

Welcome
-------

Welcome to PyICU, a Python extension wrapping IBM's International
Components for Unicode C++ library (ICU).

PyICU is a project maintained by the Open Source Applications Foundation.

The ICU homepage is: http://site.icu-project.org/


Building PyICU
--------------

Before building PyICU the ICU libraries must be built and installed. Refer
to each system's instructions for more information.

PyICU is built with distutils or setuptools:
   - verify that the ``INCLUDES``, ``LFLAGS``, ``CFLAGS`` and ``LIBRARIES``
     dictionaries in ``setup.py`` contain correct values for your platform
   - ``python setup.py build``
   - ``sudo python setup.py install``


Running PyICU
-------------

  - Mac OS X
    Make sure that ``DYLD_LIBRARY_PATH`` contains paths to the directory(ies)
    containing the ICU libs.

  - Linux & Solaris
    Make sure that ``LD_LIBRARY_PATH`` contains paths to the directory(ies)
    containing the ICU libs or that you added the corresponding ``-rpath``
    argument to ``LFLAGS``.

  - Windows
    Make sure that ``PATH`` contains paths to the directory(ies)
    containing the ICU DLLs.


What's available
----------------

See the ``CHANGES`` file for an up to date log of changes and additions.


API Documentation
-----------------

There is no API documentation for PyICU. The API for ICU is documented at
http://icu-project.org/apiref/icu4c/ and the following patterns can be
used to translate from the C++ APIs to the corresponding Python APIs.

  - strings

    The ICU string type, ``UnicodeString``, is a type pointing at a mutable
    array of ``UChar`` Unicode 16-bit wide characters. The Python unicode type
    is an immutable string of 16-bit or 32-bit wide Unicode characters.

    Because of these differences, ``UnicodeString`` and Python's ``unicode``
    type are not merged into the same type when crossing the C++ boundary.
    ICU APIs taking ``UnicodeString`` arguments have been overloaded to also
    accept Python str or unicode type arguments. In the case of ``str``
    objects, ``utf-8`` encoding is assumed when converting them to
    ``UnicodeString`` objects.

    To convert a Python ``str`` encoded in a encoding other than ``utf-8`` to
    an ICU ``UnicodeString`` use the ``UnicodeString(str, encodingName)``
    constructor.

    ICU's C++ APIs accept and return ``UnicodeString`` arguments in several
    ways: by value, by pointer or by reference.
    When an ICU C++ API is documented to accept a ``UnicodeString`` reference
    parameter, it is safe to assume that there are several corresponding
    PyICU python APIs making it accessible in simpler ways:

    For example, the
    ``'UnicodeString &Locale::getDisplayName(UnicodeString &)'`` API,
    documented at
    http://icu-project.org/apiref/icu4c/classLocale.html
    can be invoked from Python in several ways:

    1. The ICU way

        >>> from icu import UnicodeString, Locale
        >>> locale = Locale('pt_BR')
        >>> string = UnicodeString()
        >>> name = locale.getDisplayName(string)
        >>> name
        <UnicodeString: Portuguese (Brazil)>
        >>> name is string
        True                  <-- string arg was returned, modified in place

    2. The Python way

        >>> from icu import Locale
        >>> locale = Locale('pt_BR')
        >>> name = locale.getDisplayName()
        >>> name
        u'Portuguese (Brazil)'

        A ``UnicodeString`` object was allocated and converted to a Python
        ``unicode`` object.

    A UnicodeString can be coerced to a Python unicode string with Python's
    ``unicode()`` constructor. The usual ``len()``, ``str()``, comparison,
    ``[]`` and ``[:]`` operators are all available, with the additional
    twists that slicing is not read-only and that ``+=`` is also available
    since a UnicodeString is mutable. For example:

        >>> name = locale.getDisplayName()
        u'Portuguese (Brazil)'
        >>> name = UnicodeString(name)
        >>> name
        <UnicodeString: Portuguese (Brazil)>
        >>> unicode(name)
        u'Portuguese (Brazil)'
        >>> len(name)
        19
        >>> str(name)           <-- works when chars fit with default encoding
        'Portuguese (Brazil)'
        >>> name[3]
        u't'
        >>> name[12:18]
        <UnicodeString: Brazil>
        >>> name[12:18] = 'the country of Brasil'
        >>> name
        <UnicodeString: Portuguese (the country of Brasil)>
        >>> name += ' oh joy'
        >>> name
        <UnicodeString: Portuguese (the country of Brasil) oh joy>

  - error reporting

    The C++ ICU library does not use C++ exceptions to report errors. ICU
    C++ APIs return errors via a ``UErrorCode`` reference argument. All such
    APIs are wrapped by Python APIs that omit this argument and throw an
    ``ICUError`` Python exception instead. The same is true for ICU APIs
    taking both a ``ParseError`` and a ``UErrorCode``, they are both to be
    omitted.

    For example, the ``'UnicodeString &DateFormat::format(const Formattable &,
    UnicodeString &, UErrorCode &)'`` API, documented at
    http://icu-project.org/apiref/icu4c/classDateFormat.html
    is invoked from Python with:

        >>> from icu import DateFormat, Formattable
        >>> df = DateFormat.createInstance()
        >>> df
        <SimpleDateFormat: M/d/yy h:mm a>
        >>> f = Formattable(940284258.0, Formattable.kIsDate)
        >>> df.format(f)
        u'10/18/99 3:04 PM'
     
    Of course, the simpler ``'UnicodeString &DateFormat::format(UDate,
    UnicodeString &)'`` documented here:
    http://icu-project.org/apiref/icu4c/classDateFormat.html
    can be used too:

        >>> from icu import DateFormat
        >>> df = DateFormat.createInstance()
        >>> df
        <SimpleDateFormat: M/d/yy h:mm a>
        >>> df.format(940284258.0)
        u'10/18/99 3:04 PM'

  - dates

    ICU uses a double floating point type called ``UDate`` that represents the
    number of milliseconds elapsed since 1970-jan-01 UTC for dates.

    In Python, the value returned by the ``time`` module's ``time()``
    function is the number of seconds since 1970-jan-01 UTC. Because of this
    difference, floating point values are multiplied by 1000 when passed to
    APIs taking ``UDate`` and divided by 1000 when returned as ``UDate``.

    Python's ``datetime`` objects, with or without timezone information, can
    also be used with APIs taking ``UDate`` arguments. The ``datetime``
    objects get converted to ``UDate`` when crossing into the C++ layer.

  - arrays

    Many ICU API take array arguments. A list of elements of the array
    element types is to be passed from Python.

  - StringEnumeration

    An ICU ``StringEnumeration`` has three ``next`` methods: ``next()`` which
    returns a ``str`` objects, ``unext()`` which returns ``unicode`` objects
    and ``snext()`` which returns ``UnicodeString`` objects.
    Any of these methods can be used as an iterator, using the Python
    built-in ``iter`` function. 

    For example, let ``e`` be a ``StringEnumeration`` instance::

      [s for s in e] is a list of 'str' objects
      [s for s in iter(e.unext, None)] is a list of 'unicode' objects
      [s for s in iter(e.snext, None)] is a list of 'UnicodeString' objects

  - timezones

    The ICU ``TimeZone`` type may be wrapped with an ``ICUtzinfo`` type for
    usage with Python's ``datetime`` type. For example::

       tz = ICUtzinfo(TimeZone.createTimeZone('US/Mountain'))
       datetime.now(tz)

    or, even simpler::

       tz = ICUtzinfo.getInstance('Pacific/Fiji')
       datetime.now(tz)

    To get the default time zone use::

       defaultTZ = ICUtzinfo.getDefault()

    To get the time zone's id, use the ``tzid`` attribute or coerce the time
    zone to a string::

       ICUtzinfo.getInstance('Pacific/Fiji').tzid -> 'Pacific/Fiji'
       str(ICUtzinfo.getInstance('Pacific/Fiji')) -> 'Pacific/Fiji'
