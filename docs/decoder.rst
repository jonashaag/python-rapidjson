===============
 Decoder class
===============

.. module:: rapidjson

.. testsetup::

   from rapidjson import (Decoder, Encoder, DM_NONE, DM_ISO8601, DM_UNIX_TIME,
                          DM_ONLY_SECONDS, DM_IGNORE_TZ, DM_NAIVE_IS_UTC, DM_SHIFT_TO_UTC,
                          UM_NONE, UM_CANONICAL, UM_HEX, NM_NATIVE, NM_DECIMAL, NM_NAN,
                          PM_NONE, PM_COMMENTS, PM_TRAILING_COMMAS)

.. class:: Decoder(number_mode=None, datetime_mode=None, uuid_mode=None, parse_mode=None)

   :param int number_mode: enable particular behaviors in handling numbers
   :param int datetime_mode: how should :class:`datetime` and :class:`date`
                             instances be handled
   :param int uuid_mode: how should :class:`UUID` instances be handled
   :param int parse_mode: whether the parser should allow non-standard JSON extensions

   .. method:: __call__(json_str)

      :param str json_str: a string containing the ``JSON`` to be decoded
      :returns: a Python value

   .. method:: end_array(sequence)

      :param sequence: an instance implement the *mutable sequence* protocol
      :returns: a new value

      This is called, if implemented, when a *JSON array* has been completely
      parsed, and can be used replace it with an arbitrary different value:

      .. doctest::

         >>> class TupleDecoder(Decoder):
         ...   def end_array(self, a):
         ...     return tuple(a)
         ...
         >>> td = TupleDecoder()
         >>> res = td('[{"one": [1]}, {"two":[2,3]}]')
         >>> isinstance(res, tuple)
         True
         >>> res[0]
         {'one': (1,)}
         >>> res[1]
         {'two': (2, 3)}

   .. method:: end_object(mapping)

      :param mapping: an instance implementing the *mapping protocol*
      :returns: a new value

      This is called, if implemented, when a *JSON object* has been completely
      parsed, and can be used replace it with an arbitrary different value,
      like what can be done with the ``object_hook`` argument of the
      :func:`loads` function:

      .. doctest::

         >>> class Point(object):
         ...   def __init__(self, x, y):
         ...     self.x = x
         ...     self.y = y
         ...   def __repr__(self):
         ...     return 'Point(%s, %s)' % (self.x, self.y)
         ...
         >>> class PointDecoder(Decoder):
         ...   def end_object(self, d):
         ...     if 'x' in d and 'y' in d:
         ...       return Point(d['x'], d['y'])
         ...     else:
         ...       return d
         ...
         >>> pd = PointDecoder()
         >>> pd('{"x":1,"y":2}')
         Point(1, 2)

   .. method:: start_object()

      :returns: a mapping instance

      This method, when implemented, is called whenever a new *JSON object* is
      found: it must return an instance implementing the *mapping protocol*.

      It can be used to select a different implementation than the standard
      ``dict`` used by default:

      .. doctest::

         >>> from collections import OrderedDict
         >>> class OrderedDecoder(Decoder):
         ...   def start_object(self):
         ...     return OrderedDict()
         ...
         >>> od = OrderedDecoder()
         >>> print(type(od('{"foo": "bar"}')))
         <class 'collections.OrderedDict'>