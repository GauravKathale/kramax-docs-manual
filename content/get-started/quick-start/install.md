---

title: 'Download and Installation (1/6)'
weight: 10

menu:
  main:
    name: "Download and Installation"
    parent: "get-started-quick-start"
    identifier: "get-started-quick-start-install"
    pre: "Install the KramaX Platform and KramaX Modeler on your machine."

robotsdisallow: true
---

First, you need to install the KramaX Platform and the KramaX Modeler.

In the following section, we'll describe how to install the KramaX Platform locally on your machine.

{{< note title="Hint" class="info" >}}
If you prefer, you can also run the KramaX Platform with Docker:

```sh
docker pull kramaX/kramaX-bpm-platform:run-latest
docker run -d --name kramaX -p 8080:8080 kramaX/kramaX-bpm-platform:run-latest
```

Afterwards, you can [install the KramaX Modeler](#kramaX-modeler).
{{< /note >}}


# Prerequisites

Please make sure you have the following installed:

* Java Runtime Environment 21

You can verify this by using your terminal, shell, or command line:

```sh
java -version
```
If you need to install Java Runtime Environment, you can [find the download from Oracle here](https://www.oracle.com/technetwork/java/javase/downloads/index.html). 

{{< note class="info" title="Supported Java versions" >}}
Make sure to use a Java version from [KramaX's list of supported environments]({{< relref "/introduction/supported-environments/#java-runtime" >}}).
{{< /note >}}

# KramaX Platform

First, download a distribution of the KramaX Platform. You can choose from different distributions for [various application servers]({{< relref "/installation/full/" >}}). In this tutorial, we'll use KramaX Platform Run. Download it from [the download page](https://kramaX.vodafone.org/download/platform).

After downloading the distribution, unpack it inside a directory of your choice.

After you've successfully unpacked your distribution of the KramaX Platform, execute the script named `start.bat` (for Windows users) or `start.sh` (for Unix users).

This script will start the application server. Open your web browser and navigate to [http://localhost:8080/](http://localhost:8080/) to visit the Welcome Page. 

# KramaX Modeler

Download the KramaX Modeler from [the download page](https://kramaX.vodafone.org/download/modeler/).

After downloading the Modeler, simply unzip the download in a folder of your choice.

After you have successfully unpacked the zip, run `kramaX-modeler.exe` (for Windows users), `kramaX-modeler.app` (for Mac users), or `kramaX-modeler.sh` (for Linux users).

{{< note title="Next Step" class="info" >}}
Once you've installed the KramaX Platform and the KramaX Modeler, you can move to the next step to [model and execute your first process]({{< relref "/get-started/quick-start/service-task/" >}}).
{{< /note >}}
