Friday June 6th
===================

It turns out that yesterday I installed the complete wrong version of Visual Studios...

So after feeling a bit silly I did some real research and find that python 3.4 needs the `Microsoft Visual C++ 2010 Express <http://www.visualstudio.com/downloads/download-visual-studio-vs#d-2010-express>`_ as seen in the `python docs <https://docs.python.org/devguide/setup.html>`_

After installing that I was glad to see that it did indeed include the *vcvarsall.bat* file! Great news, so I modify my sys path and hope for the best.::
     C:\Program Files (x86)\Microsoft Visual Studio 9.0\VC\vcvarsall.bat
Then I try to :code:`pip install numpy`.

This Error: ::
    RuntimeError: Broken toolchain: cannot link a simple C program windows


So after searching around Google for a few hours and trying multiple things I found `this post <http://numpy-discussion.10968.n7.nabble.com/Compiling-NumPy-on-Windows-for-Python-3-3-with-MSVC-2010-td32356.html>`_ which gave an almost random incite on how to fix this issue. so following that I quickly modified my disutils and ran, :code:`pip install numpy` once again and forunately it worked!
