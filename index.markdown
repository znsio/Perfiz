---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---
[![CI Build](https://github.com/znsio/perfiz/actions/workflows/CI.yml/badge.svg)](https://github.com/znsio/perfiz/actions/workflows/CI.yml)

# Perfiz - A Dockerised API Performance Test Setup
* Run Gatling Tests (Karate Gatling at the moment) without Java on your local machine or writing any Scala simulations
* Visualize your Gatling Performance Test Metrics and Application Metrics side by side in Grafana in real-time

![Grafana Screenshot](/assets/grafana-test.png)

Jump to [quick-start](https://github.com/znsio/perfiz#quick-start--demo-5-min)

### Under the hood

Below are the projects that Perfiz leverages under the hood. Quite literally Perfiz is "Standing on the shoulders of giants".

* Gatling and Karate-Gatling
* Docker
* Grafana
* Prometheus Time Series DB
* Hooks for CAdvisor and NodeExporter

## Tutorials and Exercises

The only pre-requisite for below tutorials and exercises is **Docker** (> [version 20.10.0](https://docs.docker.com/engine/release-notes/#20100)). No other local setup required.

### Quick Start / Demo (5 min)
A project with all the pieces in place to help you try our Perfiz.
* Run a load test
* Monitor it in Grafana
* Modify load pattern etc.

Go to [Perfiz Demo](https://github.com/znsio/perfiz-demo)

### Practise Exercise (5 min)
* Learn how to integrate Perfiz into a project from scratch.
* This exercise already has sample project with REST API and related Karate API tests.
* Your task will be to leverage the Karate API tests as Perf Tests and visualize the reports in Grafana through Perfiz.

Go to [Practise Exercise](https://github.com/znsio/perfiz-demo/tree/practise-exercise)

### Running Gatling Scala Simulations (5 min)
A majority of load patterns can be described with [Perfiz YAML syntax](https://github.com/znsio/perfiz#perfiz-yaml-documentation).
Also, our preferred approach is to reuse Karate API tests as Gatling Performance Tests instead of writing Scala Simulations from scratch.

However, if you need to author Gatling Scala Simulations for advances use cases like
* Gatling [Meta DSL](https://gatling.io/docs/current/general/simulation_setup/#meta-dsl)
* Karate Gatling [Feeders](https://intuit.github.io/karate/karate-gatling/#feeders)
* OR, you have already invested in Gatling Scala Simulations

Perfiz supports running Gatling Scala Simulations with or without Karate API Tests.

Go to [Perfiz Advanced Demo](https://github.com/znsio/perfiz-demo/tree/scala_simulations_and_feeders#advanced)

### Prometheus Querying and Configuration (5 min)

* Learn how to access your PromQL expression browser to see the raw data the is being collected in Perfiz Prometheus
* Adding additional scrape configs to ```prometheus.yml``` to monitor a Java application.

Go to [Perfiz Prometheus Config](https://github.com/znsio/perfiz-demo#prometheus-configuration---adding-scrape-configs)

### Grafana Dashboards (3 min)

* Learn how to leverage [Official Community Built Dashboards](https://grafana.com/grafana/dashboards)
* Customize / Modify Dashboards and check them into version control

Go to [Perfiz Grafana Dashboards](https://github.com/znsio/perfiz-demo#grafana-dashboards---adding-jvm-dashboard)

### Karate Config (3 min)

* Learn how to refer to "karate.env" that is defined in your ```karate-config.js```
* Create multiple Perfiz config files for each of your load tests

Go to [Perfiz Karate-Config](https://github.com/znsio/perfiz-demo#setting-karateenv)

### CAdvisor Configuration - Optional

We monitor Perfiz's own container metrics through CAdvisor.
It works well on Mac OS at the moment. We are still testing Windows and Linux.
You can disable the CAdvisor setup in ```$PERFIZ_HOME/docker-compose.yml``` if this is not a priority to you.

## Perfiz Configuration YAML Documentation

Perfiz Configuration File is where we define
* Which tests / scenarios to run
* Load Pattern
* URL Patterns
* and more

You can create as many Perfiz Configuration files as you like for each setup. Example:
* Load Test for 15 min in Dev Env
* Soak Test for 2 hours in Staging Env

**Examples:**
* [perfiz-demo/perfiz-staging-load-test.yml](https://github.com/znsio/perfiz-demo/blob/main/perfiz-staging-load-test.yml)
* [perfiz-demo/perfiz-closed-loadpattern.yml](https://github.com/znsio/perfiz-demo/blob/main/perfiz-closed-loadpattern.yml)

Each of these files now allow you to codify your tests and check them in for fellow developers.

The default name for the config file is "perfiz.yml". So when you run a Perfiz Command without a config file argument it will look for 'perfiz.yml' and pick it up.

Below are all the parameters in Perfiz Config File. 

```yaml
karateFeaturesDir: #Relative Path from you repo root to the directory containing Karate Feature Files
                   #This is also the directory which contains your karate-config.js, Perfiz will make sure this file gets picked up by Karate Gatling
                   #Example: src/test/karateFeatures
karateEnv: #This sets the karate.env. Provide any env that is defined in your karate-config.js that resides in the above "karateFeaturesDir"
karateFeatures: #List of KarateFeatures which need to be run as Load Tests
  - karateFile: #Relative Path from the above karateFeaturesDir to a specific Karate Feature file
                #Example: bookings/movies/reservation.feature, if your overall directory structure is <repo-root>/src/test/karateFeatures/bookings/movies/reservation.feature
    gatlingSimulationName: #A name under which Gatling will aggregate test metrics, prefer a short descriptive name without spaces
                           #You will be able to access this in Grafana under the same name through InfluxDB DataSource
    loadPattern: #List of loads in the order that they need to be generated
      - patternType: #Modelled closely after Gatling - https://gatling.io/docs/current/general/simulation_setup/
                     #IMPORTANT: Please make sure to not mix up Open and Closed models as mentioned in above Gatling Documentation
                     #Example: "nothingFor"
                     #More Examples below
        duration: "3 seconds"
      - patternType: "rampUsers"
        userCount: "3"
        duration: "3 seconds"
      - patternType: "constantUsersPerSec"
        userCount: "3"
        duration: "3 seconds"
        randomised: "false"
      - patternType: "rampUsersPerSec"
        userCount: "3"
        targetUserCount: "6"
        duration: "3 seconds"
        randomised: "true"
    uriPatterns:
      - "/books/{isbn}" #This helps you define part in the URL that change, Gatling will capture this as a single URL with varius parameters for ISBN
      - "/book/{isbn}/author/{authorId}" #If you are familiar with karate-gatling then this is similar to Karate Protocol - https://intuit.github.io/karate/karate-gatling/#usage
  #You can keep adding more such Karate Files
  # - karateFile: test.feature
  #   gatlingSimulationName: test
  #   loadPattern:
  #     - patternType: "rampUsers"
  #     - userCount: "60"
  #     - duration: "30 minutes"
  #   and so on...
```

**IMPORTANT: Load Patterns**

Because Perfiz Leverages Gatling, it is important that we understand the [Open vs Closed Worlkload Models](https://gatling.io/docs/current/general/simulation_setup/) and **avoid mixing them**.

* Open model keywords
  * nothingFor
  * atOnceUsers
  * rampUsers
  * constantUsersPerSec
  * rampUsersPerSec
  * heavisideUsers
* Closed model keywords
  * constantConcurrentUsers
  * rampConcurrentUsers

## Developers

To create a release push an annotated tag. Example:
```shell
git tag -a <release version> -m "<release message>"
git push origin <release version>
```