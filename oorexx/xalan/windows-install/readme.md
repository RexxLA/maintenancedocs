Installing Xalan-C on Windows (new install)
===========================================

The easy way 
------------

* Download `Xerces.zip` and `Xalan.zip` from <https://sourceforge.net/projects/oorexx/files/oorexx-buildutils/Windows/1.2.0/>.
* Copy all the files in the `bin` subdirectory of `Xerces.zip` *and* all the files of the `bin` subdirectory of `Xalan.zip`
  into a new directory, say `C:\xalan`.
* Add this new directory to your path.
* Open a Command Prompt, and type `xalan` to verify that you installation works.

Now `cmake` should find the Xalan executable:

> -- XALAN_EXECUTABLE is C:/xalan-c/Xalan.exe
