# wSAST Changelog

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
