Jupyter Sphinx Extension
========================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

Jupyter Sphinx is a Sphinx extension that executes embedded code in a Jupyter
kernel, and embeds outputs of that code in the output document. It has support
for rich output such as images, Latex math and even javascript widgets.

Enabling the extension
----------------------
To enable the extension, add ``jupyter_sphinx.execute`` to your enabled extensions in
``conf.py``.

Basic Usage
-----------

You can use the ``jupyter-execute`` directive to embed code into the document::

  .. jupyter-execute::

    name = 'world'
    print('hello ' + name + '!')

The above is rendered as follows:

.. jupyter-execute::

  name = 'world'
  print('hello ' + name + '!')

Note that the code produces *output* (printing the string 'hello world!'), and the output
is rendered directly after the code snippet.

Because all code cells in a document are run in the same kernel, cells later in the document
can use variables and functions defined in cells earlier in the document:

.. jupyter-execute::

    a = 1
    print('first cell: a = {}'.format(a))

.. jupyter-execute::

    a += 1
    print('second cell: a = {}'.format(a))

Because jupyter-sphinx uses the machinery of ``nbconvert``, it is capable of rendering
much richer output than mere text; plots, for example:

.. jupyter-execute::

    import numpy as np
    from matplotlib import pyplot
    %matplotlib inline

    x = np.linspace(0, 2 * np.pi)

    pyplot.plot(x, np.sin(x) / x)
    pyplot.plot(x, np.cos(x))
    pyplot.grid()

or even full-blown javascript widgets:

.. jupyter-execute::

    import ipywidgets as w
    from IPython.display import display

    a = w.IntSlider()
    b = w.IntText()
    display(a, b)
    w.jslink((a, 'value'), (b, 'value'))

It is also possible to include code from a regular file by passing the filename as argument
to ``jupyter-execute``::

  .. jupyter-execute:: some_code.py

``jupyter-execute`` may also be used in docstrings within your Python code, and will be executed
when they are included with Sphinx autodoc.


Directive options
-----------------
You may choose to hide the code of a cell, but keep its output visible using ``:hide-code:``::

  .. jupyter-execute::
      :hide-code:

      print('this code is invisible')

produces:

.. jupyter-execute::
    :hide-code:

    print('this code is invisible')

or vice versa with ``:hide-output:``::

    .. jupyter-execute::
        :hide-output:

        print('this output is invisible')

produces:

.. jupyter-execute::
    :hide-output:

    print('this output is invisible')

You may also display the code *below* the output with ``:code-below:``::

  .. jupyter-execute::
      :code-below:

      print('this output is above the code')

produces:

.. jupyter-execute::
    :code-below:

    print('this code is below the output')


Controlling exceptions
----------------------

The default behaviour when jupyter-sphinx encounters an error in the embedded code is just to
stop execution of the document and display a stack trace. However, there are many cases where it may be
illustrative for execution to continue and for a stack trace to be shown as *output of the cell*. This
behaviour can be enabled by using the ``raises`` option::

  .. jupyter-execute::
      :raises:

      1 / 0

produces:

.. jupyter-execute::
    :raises:

    1 / 0

Note that when given no arguments, ``raises`` will catch all errors. It is also possible to give ``raises``
a list of error types; if an error is raised that is not in the list then execution stops as usual::

  .. jupyter-execute::
      :raises: KeyError, ValueError

      a = {'hello': 'world!'}
      a['jello']

produces:

.. jupyter-execute::
  :raises: KeyError, ValueError

  a = {'hello': 'world!'}
  a['jello']


Controlling the execution environment
-------------------------------------
The execution environment can be controlled by using the ``jupyter-kernel`` directive. This directive takes
the name of the Jupyter kernel in which all future cells (until the next ``jupyter-kernel`` directive) should
be run::

  .. jupyter-kernel:: python3
      :id: a_unique_name

``jupyter-kernel`` can also take a directive option ``:id:`` that names the Jupyter session;
it is used in conjunction with the ``jupyter-download`` roles described in the next section.

Note that putting a ``jupyter-kernel`` directive starts a *new* kernel, so any variables and functions declared
in cells *before* a ``jupyter-kernel`` directive will not be available in future cells.

Note that we are also not limited to working with Python: Jupyter Sphinx supports kernels for
any programming language, and we even get proper syntax highlighting thanks to the power of
Pygments.

Downloading the code as a script
--------------------------------
Jupyter Sphinx includes 2 roles that can be used to download the code embedded in a document:
``:jupyter-download:script:`` (for a raw script file) and ``:jupyter-download:notebook:`` (for
a Jupyter notebook). For example, to download all the code from this document as a script we
would use::

    :jupyter-download:script:`index`

Which produces a link like this: :jupyter-download:script:`index`. The name that the role is
applied to (``index`` in this case) is the name of the document for which you wish to download
the code. If a document contains ``jupyter-kernel`` directives with ``:id:`` specified, then
the name provided to ``:id:`` can be used to get the code for the cells belonging to the
that Jupyter session.


Configuration options
---------------------

Typically you will be using Sphinx to build documentation for a software package.

If you are building documentation for a Python package you should add the following
lines to your sphinx ``conf.py``::

    import os

    package_path = os.path.abspath('../..')
    os.environ['PYTHONPATH'] = ':'.join((package_path, os.environ.get('PYTHONPATH', '')))

This will ensure that your package is importable by any IPython kernels, as they will
inherit the environment variables from the main Sphinx process.

Here is a list of all the configuration options available to the Jupyter Sphinx extension:

jupyter_execute_default_kernel

    The default kernel to launch when executing code in ``jupyter-execute`` directives.
    The default is ``python3``.


jupyter_execute_data_priority

    The display priority of different output mimetypes. Mimetypes earlier in the data priority
    list are preferred over later ones. This is relevant if a code cell produces an output
    that has several possible representations (e.g. description text or an image).
    The default is
    ``['application/vnd.jupyter.widget-view+json', 'text/html', 'image/svg+xml', 'image/png', 'image/jpeg', 'text/latex', 'text/plain']``.


jupyter_execute_kwargs

    Keyword arguments to pass to ``nbconvert.preprocessors.execute.executenb``, which controls how
    code cells are executed. The default is ``dict(timeout=-1, allow_errors=True)``.
