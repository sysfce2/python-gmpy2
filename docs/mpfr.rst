Multiple-precision Reals
========================

gmpy2 replaces the *mpf* type from gmpy 1.x with a new *mpfr* type based on
the MPFR library. The new *mpfr* type supports correct rounding, selectable
rounding modes, and many trigonometric, exponential, and special functions. A
*context manager* is used to control precision, rounding modes, and the
behavior of exceptions.

The default precision of an *mpfr* is 53 bits - the same precision as Python's
*float* type. If the precison is changed, then ``mpfr(float('1.2'))`` differs
from ``mpfr('1.2')``. To take advantage of the higher precision provided by
the *mpfr* type, always pass constants as strings.

::

    >>> import gmpy2
    >>> from gmpy2 import mpfr
    >>> mpfr('1.2')
    mpfr('1.2')
    >>> mpfr(float('1.2'))
    mpfr('1.2')
    >>> gmpy2.get_context().precision=100
    >>> mpfr('1.2')
    mpfr('1.2000000000000000000000000000006',100)
    >>> mpfr(float('1.2'))
    mpfr('1.1999999999999999555910790149937',100)
    >>>

Contexts
--------

A *context* is used to control the behavior of *mpfr* (and *mpc*) arithmetic.
In addition to controlling the precision, the rounding mode can be specified,
minimum and maximum exponent values can be changed, various exceptions can be
raised or ignored, gradual underflow can be enabled, and returning complex
results can be enabled.

``gmpy2.context()`` creates a new context with all options set to default.
``gmpy2.set_context(ctx)`` will set the active context to *ctx*.
``gmpy2.get_context()`` will return a reference to the active context. Note
that contexts are mutable: modifying the reference returned by get_context()
will modify the active context until a new context is enabled with
set_context().

The following example just modifies the precision. The remaining options will
be discussed later.

::

    >>> gmpy2.set_context(gmpy2.context())
    >>> gmpy2.get_context()
    context(precision=53, real_prec=Default, imag_prec=Default,
            round=RoundToNearest, real_round=Default, imag_round=Default,
            emax=1073741823, emin=-1073741823,
            subnormalize=False,
            trap_underflow=False, underflow=False,
            trap_overflow=False, overflow=False,
            trap_inexact=False, inexact=False,
            trap_invalid=False, invalid=False,
            trap_erange=False, erange=False,
            trap_divzero=False, divzero=False,
            trap_expbound=False,
            allow_complex=False)
    >>> gmpy2.sqrt(5)
    mpfr('2.2360679774997898')
    >>> gmpy2.get_context().precision=100
    >>> gmpy2.sqrt(5)
    mpfr('2.2360679774997896964091736687316',100)
    >>> gmpy2.get_context().precision+=20
    >>> gmpy2.sqrt(5)
    mpfr('2.2360679774997896964091736687312762351',120)
    >>> ctx=gmpy2.get_context()
    >>> ctx.precision+=20
    >>> gmpy2.sqrt(5)
    mpfr('2.2360679774997896964091736687312762354406182',140)
    >>> gmpy2.set_context(gmpy2.context())
    >>> gmpy2.sqrt(5)
    mpfr('2.2360679774997898')
    >>> ctx.precision+=20
    >>> gmpy2.sqrt(5)
    mpfr('2.2360679774997898')
    >>> gmpy2.set_context(ctx)
    >>> gmpy2.sqrt(5)
    mpfr('2.2360679774997896964091736687312762354406183596116',160)
    >>>

Context Attributes
------------------

**precision**
    This attribute controls the precision of an *mpfr* result. The precision
    is specified in bits, not decimal digits. The maximum precision that can
    be specified is platform dependent and can be retrieved with
    **get_max_precision()**.

.. note::
    Specifying a value for precision that is too close to the maximum precision
    will cause the MPFR library to fail.

**real_prec**
    This attribute controls the precision of the real part of an *mpc* result.
    If the value is ``Default``, then the value of the precision attribute is
    used.

**imag_prec**
    This attribute controls the precision of the imaginary part of an *mpc*
    result. If the value is ``Default``, then the value of real_prec is used.

**round**
    There are five rounding modes availble to *mpfr* types:

    ``RoundAwayZero``
        The result is rounded away from 0.0.

    ``RoundDown``
        The result is rounded towards -Infinity.

    ``RoundToNearest``
        Round to the nearest value; ties are rounded to an even value.

    ``RoundToZero``
        The result is rounded towards 0.0.

    ``RoundUp``
        The result is rounded towards +Infinity.

**real_round**
    This attribute controls the rounding mode for the real part of an *mpc*
    result. If the value is ``Default``, then the value of the round attribute
    is used. Note: ``RoundAwayZero`` is not a valid rounding mode for *mpc*.

**imag_round**
    This attribute controls the rounding mode for the imaginary part of an
    *mpc* result. If the value is ``Default``, then the value of the real_round
    attribute is used. Note: ``RoundAwayZero`` is not a valid rounding mode for
    *mpc*.

**emax**
    This attribute controls the maximum allowed exponent of an *mpfr* result.
    The maximum exponent is platform dependent and can be retrieved with
    **get_emax_max()**.

**emin**
    This attribute controls the minimum allowed exponent of an *mpfr* result.
    The minimum exponent is platform dependent and can be retrieved with
    **get_emin_min()**.

.. note::
    It is possible to change the values of emin/emax such that previous *mpfr*
    values are no longer valid numbers but should either underflow to +/-0.0 or
    overflow to +/-Infinity. To raise an exception if this occurs, see
    **trap_expbound**.

**subnormalize**
    The usual IEEE-754 floating point representation supports gradual underflow
    when the minimum exponent is reached. The MFPR library does not enable
    gradual underflow by default but it can be enabled to precisely mimic the
    results of IEEE-754 floating point operations.

**trap_underflow**
    If set to ``False``, a result that is smaller than the smallest possible
    *mpfr* given the current exponent range will be replaced by +/-0.0. If set
    to ``True``, an ``UnderflowError`` exception is raised.

**underflow**
    This flag is not user controllable. It is automatically set if a result
    underflowed to +/-0.0 and trap_underflow is ``False``.

**trap_overflow**
    If set to ``False``, a result that is larger than the largest possible
    *mpfr* given the current exponent range will be replaced by +/-Infinity. If
    set to ``True``, an ``OverflowError`` exception is raised.

**overflow**
    This flag is not user controllable. It is automatically set if a result
    overflowed to +/-Infinity and trap_overflow is ``False``.

**trap_inexact**
    This attribute controls whether or not an ``InexactError`` exception is
    raised if an inexact result is returned. To check if the result is greater
    or less than the exact result, check the **rc** attribute of the *mpfr*
    result.

**inexact**
    This flag is not user controllable. It is automatically set if an inexact
    result is returned.

**trap_invalid**
    This attribute controls whether or not an ``InvalidOperationError``
    exception is raised if a numerical result is not defined. A special
    NaN (Not-A-Number) value will be returned if an exception is not raised.
    The ``InvalidOperationError`` is a sub-class of Python's ``ValueError``.

    For example, ``gmpy2.sqrt(-2)`` will normally return *mpfr('nan')*.
    However, if allow_complex is set to ``True``, then an *mpc* result will
    be returned.

**invalid**
    This flag is not user controllable. It is automatically set if an invalid
    (Not-A-Number) result is returned.

**trap_erange**
    This attribute controls whether or not a ``RangeError`` exception is raised
    when certain operations are performed on NaN and/or Infinity values.
    Setting trap_erange to ``True`` can be used to raise an exception if
    comparisons are attempted with a NaN.

    ::

        >>> gmpy2.set_context(gmpy2.get_context())
        >>> mpfr('nan') == mpfr('nan')
        False
        >>> gmpy2.get_context().trap_erange=True
        >>> mpfr('nan') == mpfr('nan')
        Traceback (most recent call last):
          File "<stdin>", line 1, in <module>
        gmpy2.RangeError: comparison with NaN
        >>>

**erange**
    This flage is not user controllable. It is automatically set if an erange
    error occurred.

**trap_divzero**
    This attribute controls whether or not a ``DivisionByZeroError`` exception
    is raised if division by 0 occurs. The ``DivisionByZeroError`` is a
    sub-class of Python's ``ZeroDivisionError``.

**divzero**
    This flag is not user controllable. It is automatically set if a division
    by zero occurred and NaN result was returned.

**trap_expbound**
    This attribute controls whether or not an ``ExponentOutOfBoundsError``
    exception is raised if exponents in an operand are outside the current
    emin/emax limits.

**allow_complex**
    This attribute controls whether or not an *mpc* result can be returned if
    an *mpfr* result would normally not be possible.


Context Methods
---------------

**clear_flags()**
    Clear the underflow, overflow, inexact, invalid, erange, and divzero flags.
