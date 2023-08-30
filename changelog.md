# wSAST Changelog

## wSAST v0.1-alpha (release date 25-08-2023)

* Added a simple JSP-to-Java source processor
* Fixed an infinite loop parsing java _Patterns_
* Fixed a null pointer exception in the Common Rules Engine simple variable rule handling
* Fixed a null pointer exception pruning variables from scope
* Fixed a bug whereby class members may not have always have a scope attached
* Fixed a complexity issue in Common Rules data rule processing and added regex timeout
* Fixed a bug parsing java _record_ types nested within classes
* Fixed a bug parsing certain Java method references
* Fixed a bug in parse aborting and added parse timeout setting for Java parser
* Fixed a bug in Java translation to WSIL where for loop update was placed outside of loop logic
* Fixed a stack overflow exception in `graph calls --filter-root` with recursive functions
* Fixed a bug in Common Rules Engine simple subscribers when source/sink were same expression
* Changed Common Rules Engine simple variable sources to reduce false positives
* Added a lock around Common Rules Engine simple subscriber file writing
* Suppressed an error parsing annotation type definitions
* Improved Java _record_ support (convert _RecordHeader_ to WSIL constructor)
* Improved handling of Java static imports
* Improved Java thread scheduling to allow for faster multi-file parse times

## wSAST v0.1-alpha (release date 07-08-2023)
* **Modified commonrules.xml to remove generic function rules by default as these produce a great volume of false positives - re-enable as needed.**
* Fixed bug in `graph calls` feature which reversed the logic for the `--inclusive` flag.
* Fixed bug in `paths` command which omitted a step in certain paths.
* Fixed null pointer crash in Java parser handling module-info.java (now omitted from analysis).
* Fixed support for Java interfaces containing classes (now promoted to WSIL classes).
* Fixed bug where certain Java contextual keywords forbidden as identifiers were valid.
* Fixed bug in `calls` feature where some calls from recursive functions were not listed.
* Fixed exception in Java parser where `super` referenced missing super class.
* Fixed a licensing issue affecting hosts running certain AV/EDR products.
* Fixed MotW error ("Error loading assembly '<name'>: Could not load file or assembly\[...\] HRESULT 0x80131515") after using Windows ZIP for extraction.
* Fixed path supression issue that could reduce accuracy of certain dataflow scans.
* Fixed bug in class graphing which occasionally introduced spurious links.
* Fixed null pointer bug in CommonRulesEngine matching variable prefixes.
* Added a feature to list all calls from matching roots `--filter-root`.
* Added `--ignorecase` flag to all regex filters.
* Added support for annotations to Java parser (for future annotation-based rules).
* Added simple progress marker when analysis runs long over large codebase.
* Added update file erasure on wSAST.exe launch.
* Tested more thoroughly on Windows 11, Server 2016.

## wSAST v0.1-alpha (release date 03-07-2023)

* Initial release.
