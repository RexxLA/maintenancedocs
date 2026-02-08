Xalan
=====

From [The Apache Xalan Project](https://xalan.apache.org/) "The Apache Xalan Project develops and maintains libraries and programs that transform XML documents using XSLT standard stylesheets".

When Xalan is installed, the ooRexx build process uses it and a set of XSL templates to automatically generate a set of files.

The following information was provided by JLF.

See [this thread](https://sourceforge.net/p/oorexx/mailman/oorexx-devel/thread/61d396bb-8a09-473b-8e24-b50578f01c47@gmail.com/) (20251023) about Xalan in the oorexx-devel list.

Xalan for Windows can be downloaded [here](https://sourceforge.net/projects/oorexx/files/oorexx-buildutils/Windows/1.2.0/).

main/trunk/CMakeLists.txt contains the macros used to generate the files (look for `"if (XALAN_EXECUTABLE)"`)

From main/trunk/interpreter/messages/rexxmsg.xml, the following files are generated (see macro generate_message_file):

* main/trunk/interpreter/messages/RexxErrorCodes.h (template: main/trunk/interpreter/messages/RexxErrorCodes.xsl)
* main/trunk/interpreter/messages/RexxMessageNumbers.h (template: main/trunk/interpreter/messages/RexxMessageNumbers.xsl)
* main/trunk/interpreter/messages/RexxMessageTable.h   (template: main/trunk/interpreter/messages/RexxMessageTable.xsl)
* main/trunk/interpreter/messages/RexxErrorMessages.h  (template: main/trunk/interpreter/messages/RexxErrorMessages.xsl)
* main/trunk/api/oorexxerrors.h                        (template: main/trunk/interpreter/messages/ApiErrorCodes.xsl)
* main/trunk/interpreter/messages/errnums.xml          (template: main/trunk/interpreter/messages/errnums.xsl)
* main/trunk/interpreter/messages/errnumsrxqueue.xml   (template: main/trunk/interpreter/messages/errnumsrxqueue.xsl)
* main/trunk/interpreter/messages/errnumssubcom.xml    (template: main/trunk/interpreter/messages/errnumssubcom.xsl)
* main/trunk/interpreter/messages/errnumsrexxc.xml     (template: main/trunk/interpreter/messages/errnumsrexxc.xsl)

From main/trunk/interpreter/behaviour/PrimitiveClasses.xml, the following files are generated (see macro generate_class_header):

* main/trunk/interpreter/behaviour/PrimitiveBehaviourNames.h   (template: main/trunk/interpreter/behaviour/PrimitiveBehaviourNames.xsl)
* main/trunk/interpreter/behaviour/PrimitiveBehaviours.cpp     (template: main/trunk/interpreter/behaviour/PrimitiveBehaviours.xsl)
* main/trunk/interpreter/behaviour/VirtualFunctionTable.cpp    (template: main/trunk/interpreter/behaviour/VirtualFunctionTable.xsl)
* main/trunk/interpreter/behaviour/ClassTypeCodes.h            (template: main/trunk/interpreter/behaviour/ClassTypeCodes.xsl)

These appear to be no longer used:

* main/trunk/interpreter/platform/unix/UnixGencat.xsl              
* main/trunk/interpreter/platform/windows/WinMessageResource.xsl   
