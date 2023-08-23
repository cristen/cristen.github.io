---
layout: post
title: Generate your Sphinx documentation
date: 2022-12-04 16:42
category: Python
tags: ["programming", "python", "documentation"]
---

# How to create a documentation with Sphinx

1. Change into your project directory. E.g., cd ~/my_repos/my_project. If this is a typical Python project, there will likely be a sub-directory in that directory that echoes your project name (where the bulk of your source lives). This is just a note for the instructions further down.

2. Make a directory for the documentation to be generated into: 
```console
mkdir docs
```
3. Switch to that directory: 
```console
cd docs
```
4. Invoke Sphinx Quick Start: 
```console
sphinx-quickstart
```
5. The basics have been created for you. You still have some work to do. Using the text editor of your choice, open the `docs/source/conf.py` file and make the following changes:
   1. Near the top of the file, uncomment the following lines:
   
        ```python

        import os
        import sys
        # change this
        sys.path.insert(0, os.path.abspath('.'))
        # to that
        sys.path.insert(0, os.path.abspath('../..'))
        # Add the extensions
        extensions = [
            'sphinx.ext.autodoc',
            'sphinx.ext.viewcode',
            'sphinxcontrib.napoleon'
        ]
        ```
   3. Save the file and quit your editor.

6. Generate the index .rst files: 

    ```console
    sphinx-apidoc -f -o source/ ../
    ```

7. Finally generate the HTML

    ```console
    make html
    # or
    ./make.bat html
    ```

And voilà, you have a good starting point for your projects' documentation.