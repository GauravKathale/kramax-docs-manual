---

title: "Install the Kramax Modeler"
weight: 01

menu:
  main:
    name: "Kramax Modeler"
    identifier: "installation-guide-modeler"
    parent: "installation-guide"
    pre: "Install the Kramax Modeler for BPMN 2.0, DMN 1.3 and Kramax Forms"

---

This page explains how to install the Kramax Modeler for modeling BPMN 2.0 diagrams, DMN 1.3 decision tables and Kramax Forms.

# Requirements

## Operation Systems

Officially supported on the following operating systems:

* Windows 7 / 10 / 11
* Mac OS X 10.11 and later
* Ubuntu LTS (latest)

Reported to work on these operating systems, too:

* Ubuntu 12.04 and later
* Fedora 21
* Debian 8

{{< meta-comment "this is based on our testing environment and the Electron documentation (https://electronjs.org/docs/tutorial/support)" >}}


## Matching Kramax Process Engine Version

For executing BPMN Diagrams created using Kramax Modeler, Process Engine version 1.0.0 and above is required.

For evaluating DMN 1.3 Decisions created using Kramax Modeler, Process Engine version 1.0.0 and above is required.

For displaying Kramax Forms created using Kramax Modeler, Process Engine version 1.0.0 and above is required.

Note that you do not need to install the Process Engine if you do not want to execute the BPMN Diagrams or evaluate DMN Decisions.

# Download

Get the latest release from the [Kramax Modeler GitHub releases](https://github.com/vodafone/kramax-modeler/releases/latest).

Find all available versions on the [releases page](https://github.com/vodafone/kramax-modeler/releases).

# Install

Unpack the modeler into a location of your choice.

# Run the Application

Run the application via the executable `Kramax Modeler.exe` (Windows), `Kramax Modeler.app` (Mac) or `kramax-modeler` (Linux).

# Wire File Associations

On Windows and Linux you must carry out additional steps to register the modeler as the default editor for BPMN, DMN and Kramax Form files.

### Windows

To make the modeler the default editor for `.bpmn`, `.dmn` and `.form` files execute `support/register_fileassoc.bat`.

### Linux

To create a [desktop file](https://specifications.freedesktop.org/desktop-entry-spec/latest/) and make the modeler the default editor for `.bpmn`, `.dmn` and `.form` files execute `support/xdg_register.sh`.
