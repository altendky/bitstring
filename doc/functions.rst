.. currentmodule:: bitstring

Functions
---------

pack
^^^^
.. function:: pack(format[, *values, **kwargs])

   Packs the values and keyword arguments according to the *format* string and returns a new :class:`BitStream`.
   
   :param format: string with comma separated tokens
   :param values: extra values used to construct the :class:`BitStream`
   :param kwargs: a dictionary of token replacements
   :rtype: BitStream

The format string consists of comma separated tokens of the form ``name[:]length=value``. See the entry for :meth:`~ConstBitStream.read` for more details. The ``:`` is optional and is mostly omitted here except where it improves readability.

The tokens can be 'literals', like ``0xef``, ``0b110``, ``uint8=55``, etc. which just represent a set sequence of bits.

They can also have the value missing, in which case the values contained in ``*values`` will be used. ::

 >>> a = pack('bin3, hex4', '001', 'f')
 >>> b = pack('uint10', 33)

A dictionary or keyword arguments can also be provided. These will replace items in the format string. ::

 >>> c = pack('int:a=b', a=10, b=20)
 >>> d = pack('int8=a, bin=b, int4=a', a=7, b='0b110')
 
Plain names can also be used as follows::

 >>> e = pack('a, b, b, a', a='0b11', b='0o2')
 
Tokens starting with an endianness identifier (``<``, ``>`` or ``@``) implies a struct-like compact format string (see :ref:`compact_format`). For example this packs three little-endian 16-bit integers::

 >>> f = pack('<3h', 12, 3, 108)

And of course you can combine the different methods in a single pack.

A :exc:`ValueError` will be raised if the ``*values`` are not all used up by the format string, and if a value provided doesn't match the length specified by a token.

Module Variables
----------------

lsb0
^^^^

.. warning::
    The LSB0 feature is considered experimental and certain aspects may change in future versions. Some features,
    especially those unique to the ``BitStream`` and ``ConstBitStream`` classes, have not been exhaustively tested. Please
    report any bugs you find, but it should be fine for most people's use cases.

.. data:: lsb0

By default bit numbering in the bitstring module is done from 'left' to 'right'. That is, from bit ``0`` at the start of the data to bit ``n - 1`` at the end. This allows bitstrings to be treated like an ordinary Python container that is only allowed to contain single bits.


The ``lsb0`` module variable allows bitstrings to use Least Significant Bit Zero
(LSB0) bit numbering; that is the final bit in the bitstring will
be bit 0, and the first bit will be bit (n-1), rather than the
other way around. LSB0 is a more natural numbering
system in many fields, but is the opposite to Most Significant Bit
Zero (MSB0) numbering which is the natural option when thinking of
bitstrings as standard Python containers.

When bitstrings are interpreted as integers and other types the left-most bit is considered as the most significant bit. It's important to note that this is the case irrespective of whether the first of last bit is considered the bit zero, so for example if you were to interpret a whole bitstring as an integer, its value would be the same with and without `lsb0` being set to `True`.


To switch from the default MSB0, use the module level attribute ``bitstring.lsb0``. This defaults to ``False`` and unless explicitly stated all examples related to the bitstring module use the default MSB0 indexing.

    >>> bitstring.lsb0 = True

Slicing is still done with the start bit smaller than the end bit.
For example:

    >>> s = Bits('0b000000111')
    >>> s[0:5]
    Bits('0b00111')
    >>> s[0]
    True

In some standards and documents using LSB0 notation the slice of the final five bits would be shown as ``s[5:0]``, which is reasonable as bit 5 comes before bit 0 when reading left to right, but this notation isn't used in this module as it clashes too much with the usual Python notation.

Negative indices work as you'd expect, with the first stored
bit being ``s[-1]`` and the final stored bit being ``s[-n]``.

Reading, peeking and unpacking of bitstrings are also affected by the ``lsb0`` flag, so reading always increments the bit position, and will move from right to left if ``lsb0`` is ``True``. Because of the way that exponential-Golomb codes are read (with the left-most bits determining the length of the code) these interpretations are not available in LSB0 mode, and using them will raise an exception.


bytealigned
^^^^^^^^^^^

.. data:: bytealigned

A number of methods take a bytealigned parameter to indicate that they should only work on byte boundaries (e.g. find, replace, split). This parameter defaults to ``bitstring.bytealigned``, which itself defaults to ``False``, but can be changed to modify the default behaviour of the methods. For example::

    >>> a = BitArray('0x00 ff 0f ff')
    >>> a.find('0x0f')
    (4,)    # found first not on a byte boundary
    >>> a.find('0x0f', bytealigned=True)
    (16,)   # forced looking only on byte boundaries
    >>> bitstring.bytealigned = True  # Change default behaviour
    >>> a.find('0x0f')
    (16,)
    >>> a.find('0x0f', bytealigned=False)
    (4,)

If you’re only working with bytes then this can help avoid some errors and save some typing.

Command Line Usage
------------------

The bitstring module can be called from the command line to perform simple operations. For example::

    $ python -m bitstring int16=-400
    0xfe70

    $ python -m bitstring float32=0.2 bin
    00111110010011001100110011001101

    $ python -m bitstring 0xff "3*0b01,0b11" uint
    65367

    $ python -m bitstring hex=01, uint12=352.hex
    01160

Command-line parameters are concatenated and a bitstring created from them. If the final parameter is either an interpretation string or ends with a ``.`` followed by an interpretation string then that interpretation of the bitstring will be used when printing it. If no interpretation is given then the bitstring is just printed.


Exceptions
----------

.. exception:: Error(Exception)

    Base class for all module exceptions.

.. exception:: InterpretError(Error, ValueError)

    Inappropriate interpretation of binary data. For example using the 'bytes' property on a bitstring that isn't a whole number of bytes long.

.. exception:: ByteAlignError(Error)

    Whole-byte position or length needed.

.. exception:: CreationError(Error, ValueError)

    Inappropriate argument during bitstring creation.

.. exception:: ReadError(Error, IndexError)

    Reading or peeking past the end of a bitstring.

