===================
Argument Processing
===================

``cmd2`` makes it easy to add sophisticated argument processing to your commands using the ``argparse`` python module.
``cmd2`` handles the following for you:

1. Parsing input and quoted strings like the Unix shell
2. Parse the resulting argument list using an instance of ``argparse.ArgumentParser`` that you provide
3. Passes the resulting ``argparse.Namespace`` object to your command function
4. Adds the usage message from the argument parser to your command.
5. Checks if the ``-h/--help`` option is present, and if so, display the help message for the command

These features are all provided by the ``@with_argument_parser`` decorator.

Using the decorator
===================

For each command in the ``cmd2`` subclass which requires argument parsing,
create an instance of ``argparse.ArgumentParser()`` which can parse the
input appropriately for the command. Then decorate the command method with
the ``@with_argument_parser`` decorator, passing the argument parser as the
first parameter to the decorator. Add a third variable to the command method, which will contain the results of ``ArgumentParser.parse_args()``.

Here's what it looks like::

      argparser = argparse.ArgumentParser()
      argparser.add_argument('-p', '--piglatin', action='store_true', help='atinLay')
      argparser.add_argument('-s', '--shout', action='store_true', help='N00B EMULATION MODE')
      argparser.add_argument('-r', '--repeat', type=int, help='output [n] times')
      argparser.add_argument('word', nargs='?', help='word to say')

      @with_argument_parser(argparser)
      def do_speak(self, argv, opts)
         """Repeats what you tell me to."""
         arg = opts.word
         if opts.piglatin:
            arg = '%s%say' % (arg[1:], arg[0])
         if opts.shout:
            arg = arg.upper()
         repetitions = opts.repeat or 1
         for i in range(min(repetitions, self.maxrepeats)):
            self.poutput(arg)

.. note::

   The ``@with_argument_parser`` decorator sets the ``prog`` variable in
   the argument parser based on the name of the method it is decorating.
   This will override anything you specify in ``prog`` variable when
   creating the argument parser.


Help Messages
=============

By default, cmd2 uses the docstring of the command method when a user asks
for help on the command. When you use the ``@with_argument_parser``
decorator, the formatted help from the ``argparse.ArgumentParser`` is
appended to the docstring for the method of that command. With this code::

   argparser = argparse.ArgumentParser()
   argparser.add_argument('tag', nargs=1, help='tag')
   argparser.add_argument('content', nargs='+', help='content to surround with tag')
   @with_argument_parser(argparser)
   def do_tag(self, cmdline, args=None):
      """create a html tag"""
      self.stdout.write('<{0}>{1}</{0}>'.format(args.tag[0], ' '.join(args.content)))
      self.stdout.write('\n')

The ``help tag`` command displays:

.. code-block:: none

   create a html tag
   usage: tag [-h] tag content [content ...]

   positional arguments:
     tag         tag
     content     content to surround with tag

   optional arguments:
     -h, --help  show this help message and exit


If you would prefer the short description of your command to come after the usage message, leave the docstring on your method empty, but supply a ``description`` variable to the argument parser::

   argparser = argparse.ArgumentParser(description='create an html tag')
   argparser.add_argument('tag', nargs=1, help='tag')
   argparser.add_argument('content', nargs='+', help='content to surround with tag')
   @with_argument_parser(argparser)
   def do_tag(self, cmdline, args=None):
      self.stdout.write('<{0}>{1}</{0}>'.format(args.tag[0], ' '.join(args.content)))
      self.stdout.write('\n')

Now when the user enters ``help tag`` they see:

.. code-block:: none

   usage: tag [-h] tag content [content ...]

   create an html tag

   positional arguments:
     tag         tag
     content     content to surround with tag

   optional arguments:
     -h, --help  show this help message and exit


To add additional text to the end of the generated help message, use the ``epilog`` variable::

   argparser = argparse.ArgumentParser(
      description='create an html tag',
      epilog='This command can not generate tags with no content, like <br/>.'
   )
   argparser.add_argument('tag', nargs=1, help='tag')
   argparser.add_argument('content', nargs='+', help='content to surround with tag')
   @with_argument_parser(argparser)
   def do_tag(self, cmdline, args=None):
      self.stdout.write('<{0}>{1}</{0}>'.format(args.tag[0], ' '.join(args.content)))
      self.stdout.write('\n')

Which yields:

.. code-block:: none

   usage: tag [-h] tag content [content ...]

   create an html tag

   positional arguments:
     tag         tag
     content     content to surround with tag

   optional arguments:
     -h, --help  show this help message and exit

   This command can not generate tags with no content, like <br/>


Deprecated optparse support
===========================

The ``optparse`` library has been deprecated since Python 2.7 (released on July
3rd 2010) and Python 3.2 (released on February 20th, 2011). ``optparse`` is
still included in the python standard library, but the documentation
recommends using ``argparse`` instead.

``cmd2`` includes a decorator which can parse arguments using ``optparse``. This decorator is deprecated just like the ``optparse`` library.

Here's an example::

   opts = [make_option('-p', '--piglatin', action="store_true", help="atinLay"),
           make_option('-s', '--shout', action="store_true", help="N00B EMULATION MODE"),
           make_option('-r', '--repeat', type="int", help="output [n] times")]

   @options(opts, arg_desc='(text to say)')
   def do_speak(self, arg, opts=None):
     """Repeats what you tell me to."""
     arg = ''.join(arg)
     if opts.piglatin:
         arg = '%s%say' % (arg[1:], arg[0])
     if opts.shout:
         arg = arg.upper()
     repetitions = opts.repeat or 1
     for i in range(min(repetitions, self.maxrepeats)):
         self.poutput(arg)


The optparse decorator performs the following key functions for you:

1. Use `shlex` to split the arguments entered by the user.
2. Parse the arguments using the given optparse options.
3. Replace the `__doc__` string of the decorated function (i.e. do_speak) with the help string generated by optparse.
4. Call the decorated function (i.e. do_speak) passing an additional parameter which contains the parsed options.