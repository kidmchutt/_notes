# PlanTest

## High-level Usage description

Graphical application for test planning. All elements to test should be described in the application, based on the [page model](#page_model_desc), displayed in the GUI in the style of a mind map/dev diagram, which could then be used as mapping model through all the application tested to ensure all the usage scenarios are passed through and all the validations and elements states are correct.

It should create all the passthroughs semi-automatically, final stage should be the map and test cases with all the settings defined in the preparation step and all the results for the tests itself will end as set of human readable information which can then be used as test protocol for the proper app version.

The application itself will be developed in the python language, using FOSS tools, it will be multi-platform (Linux, Windows) and it will support multiple languages (English and Czech in the first release) using language file in human-readable format or depending on the used technology (if Qt will be used).

There will be database with the used elements, tests and results, different for each test project. Test cases and the results will be exportable to json, xml and markdown format. 

In the future versions, test cases will be importable back, used as backup

## User workflows

### CLI Applications or services call
- User will create or open project for the application to test.
- User will create element descripting basic CLI/Service call (Application to Test)
	- ex. name of the CLI application used for testing defining its name and link - name ```testCLI```, link with full address ```.\testCLI.exe```
- User will create elements with the calls to test list and the expected result in the command line/bash. This can use test data variables which will be defined as well in their specific element (Test Data)
	- ex. for the ```.\testCLI.exe help``` parameter the element will contain simple ```help``` and the expected result either in text description or ready to be automated - so the concrete return value.
	- ex. for the ```.\testCLI.exe login --user $usr1 --password $usr1_pwd``` both of the variables will needs to be defined in the (Test Data) part of the tests.
	- ex. for more complex parameters, parameter can be defined beforehand and user will define in test cases which parameter should be called. Parameters can have set mandatory flag.
- User will create element with the test data used in the tests
	- it could be basic list containing name and value used in the previous step example - ex. ```$usr1 = 'Test user 1'``` and ```$usr1_pwd = 'passphrase'```
	- or complex test data list combining simple approach and hierarchical  data, which will be then called by both their subset name and variable name - ex. ```$login.$usr```. For better understanding, not necessary better readibility, simple values could be called as ```$default.$usr1``` of the variable is not part of the hierarchy. For the sake of this functionality ```$default``` word is reserved.

### Applications with GUI
- User will create or open project for the application to test.
- User will define the screens tested app (or pages for webpage) is using by identifiying all the elements to test - links, titles, buttons, inputs, etc.
	- this might be done manually in the app 
		- adding either screen description or
		- adding CLI commands list with possible options and information if they are mandatory 
	- or uploaded using toml/json/xml file with data (see [pre-defined objects](#pre_defined_objects))
	- this should contain default state of the elements, possible error messages and it's behavior
- (GUI) User will link active elements (buttons, links, checkboxes, etc.) to behavior - if the clicking will open new window/page, change something on the page based on the values on the page and/or input test data.
- User will define test data or test data ranges. Simple app with form to fill, let's say name, surname, age, address - this might be test data, but they might need to be linked to some database or sets to be used semi-randomly.
	- this might be done manually in the app 
	- or uploaded using toml/json/xml file with data
	- there might be more than one set of the test data for the tests	
- Then the app will detect all the passthroughs to test all the elements on all connected pages, all the workflows and generate set of test scenarios - both happy day (Test to Success) and failing (Test to Fail) to test error messages and fails.
- Test set will be saved then in a database, versioned and updated if necessary.
	- User can set priorities for each test.
	- This test set will be then exportable - as a xls file for manual testing (basic) or as a data set to be uploaded to some app for test runs (Spira Test, Testrail, etc.), which can be defined manually.

In some later version, we will add test runs management and test run protocols export.

## Tools
- SQL - supported databases should be:
	- SQLite (small projects, standalone)
	- MSSQL (robust)
	- Oracle?
	- MariaDb?

## Todos
- [ ] decide language format used (Qt?)
- [ ] decide GUI used (Qt?)
- [ ] create database schema with the emphasis on extension
	- [ ] table with app version (used for db update) and project name
	- [ ] table of elements
	- ?[ ] table of screens
- [ ] New Page - this will need more elaboration and thinking

## Page model objects

### Pre-defined objects {#pre_defined_objects}

All the objects used in the tested project are saved in the proper elements.toml file. Here are some pre-defined objects, user can add more objects to the file if necessary.
All the "element_" variables are mandatory, these are used for element recognition. Other values are not mandatory and they will be used as valued for testing. elements.toml file is used once when the test project is being set, other elements or test values for the elements will be set while the program runs only. This should help with project database integrity.

Basic elements.toml file is described here, default values are set in the example, defining variable types:

```toml
[elements]

[elements.button]
element_name = "button"
element_id = ""
value = ""
visible = true
disabled = false

[elements.checkbox]
element_name = "checkbox"
element_id = ""
checked = false
visible = true
disabled = false

[elements.combo]
element_name = "combo"
element_id = ""
values = [""]
visible = true
disabled = false

[elements.input]
element_name = "input"
element_id = ""
value = ""
single-line = true
visible = true
disabled = false
validation = "" #regexp expected

[elements.image]
element_name = "image"
element_id = ""
value = ""
visible = true

[elements.list]
element_name = "list"
element_id = ""
values = [""]
selected_value = ""
visible = true
disabled = false
```

This elements.toml file is used once when the project is created. After that, everything will be saved in the database and updating this file will not be transferred to application, but it will need to be done in the application itself.

Pages are then defined by list of elements and subpages and behavior of the elements on the page (if checkbox is checked, input will be visible and set to enabled, etc.).

## Usage

### Menu items
- Project
	- Open project
	- [New project](#window_new_project)
	- [Project settings](#window_project_settings)
	- Close project and quit
- Elements
	- [New element](#window_element) Ctrl-N
	- [Elements list](#window_list_elements)
- Pages
	- [New page](#window_page) Ctrl-P
	- [Pages list](#window_list_page)
- About
	- About the application
	- Settings

### Dialogs
Not all dialogs are described here, only those which are not simple self-explanatory.

#### New Project {#window_new_project}
Dialog for creating new project.

All the fields are mandatory
- input Project name
- input Project database prefilled name and location
- input with link to the elements.toml file - prefilled with the default elements.toml location
- buttons Create and Cancel

Create will create database with prepared elements. From now on, all the elements and pages must be created manually from the menu.

#### Project settings {#window_project_settings}
Dialog for project settings.

All the fields are mandatory
- input Project name (filled)
- input Project database (filled)
- buttons Update and Cancel

Pressing Update will open modal confirmation.

#### New Element {#window_element}
Dialog for creating and editing elements. Default values are loaded only if element does not exist in the database, otherwise all the values are filled from database.

Mandatory fields:
- element_name (string)

Non-mandatory fields:
- element_id (string)
- value (string)
- visible (bool) - default true
- disabled (bool) - default false
- table with three fields:
	- field name (string, no spaces allowed)
	- type (list: string, bool, number)
	- default value
- buttons Save and Cancel

#### List of Elements {#window_list_elements}
Dialog with all prepared elements. Elements are listed here, can be edited and set active or non active pro the project.

Elements are listed in the table, see example:

|Element_name|Active|
|--|--|
|Button|yes|
|Input|no|

Clicking on value in column Active will change the value and save the change to database.
Clicking on value in column Element_name will open dialog New Element {#window_element} with prefilled values loaded from database.

#### New Page {#window_page}
Dialog to create or edit page or subpage.

Mandatory fields:
- page_name (string)
- page_id (string, no spaces allowed)

Non-mandatory fields:
- visible (bool)
- editable (bool)
- table with all the elements on the page (see example)

|element_name|element_id|tab_order|visible|disabled|value|...|
|-|-|-|-|-|-|-|
|button|CSS:"Hello"|1|true|false|Hello World|-|
|special|CSS:"Special_element"|2|true|false||spec;bool;true|

#### List of Pages {#window_list_page}
Dialog to list created pages and subpages.

Pages and subpages are listed here, see example:

|Page_name|Active|
|--|--|
|About|yes|
|Landing_page|yes|

Clicking on value in column Active will change the value and save the change to database.
Clicking on value in column Page_name will open dialog New Element {#window_page} with prefilled values loaded from database.

### Setting up the project workflow

1. Select "New project" from the menu.
2. Dialog ["New project"](#window_new_project) will open. Fill the mandatory values and save. This should create new project database with prepared structure and elements.

## Definitions

### Page model {#page_model_desc}

Page model description is based on the GUI representation of the testes application, defines "pages" or "screens" of the application, describes what user see and what can they work with. It uses pre-defined set of objects, like button, input, checkbox, combobox, list, etc. The basic set is pre-defined in the application, it can be expanded - both by new elements and the values elements are using.

For examples, see [pre-defined objects](#pre_defined_objects).

## Notes
- use the [database memory](https://github.com/radimsem/remindb) for AI agent
- each method and function must have unit tests
- SQL migration scripts - both standalone and as a part of the installation
- documentation before development, use mermaid instead of text representing images in the docs
- the whole project should use FOSS/MIT 3rd party libraries