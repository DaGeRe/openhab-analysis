# OpenHab Analysis

This repo shows how to use Peass to analyze OpenHab repos. The main purpose is to visualize unit test call graphs using ExplorViz.

This has been done using two repos:
- https://github.com/openhab/org.openhab.binding.zigbee.git (Smaller, but less to see)
- https://github.com/openhab/openhab-core.git (Bigger, analysis are more time-consuming)

## Prerequisites

- Build Kieker: `git clone https://github.com/kieker-monitoring/kieker.git && cd kieker && ./gradlew assemble -x test -x check -x apidoc`
- Build Peass: `git clone -b develop https://github.com/DaGeRe/peass.git && cd peass && ./mvnw clean package -DskipTests -P buildStarter`
- Get ExplorViz: `git clone https://github.com/ExplorViz/deployment.git`

## Regression Test Selection

Peass does a prior analysis step before visualizations are possible. This is done using
- `./peass select -folder ../projects/org.openhab.binding.zigbee/ -startcommit 2f0a43ef8ddcced53b681e1a761c5cfde99ac562 -endcommit a8b7f0a22067a2f33503974050a2fe1eba49700c &> select-zigbee-full.txt`
- `./peass select -folder ../projects/openhab-core/ -startcommit 6764b6d94ce054a5976731781a9bc1141cd53c7b -endcommit cd26f6feaf2a4fda3d2567b151d1e5effffa878f &> select-openhab-core-full.txt`

Getting these results is time-consuming. Therefore, the results of these steps for the given commits are in this repo. If you want more / different commits, you'll need to re-execute this steps.

## Starting ExplorViz Visualization

To start the visualization, we need to start ExplorViz, start the Kieker-Otel-Converter, and start the Peass-automation for starting the unit tests with trace export to Kieker. This is done using the following steps:
- Start ExplorViz: `cd deployment && docker compose up -d`
- Start the Kieker-Otel-Transformer: `cd $KIEKER_HOME/tools/otel-transformer/build/distributions && unzip otel-transformer-2.0.0-SNAPSHOT.zip && otel-transformer-2.0.0-SNAPSHOT/bin/otel-transformer -lp 10001 --configuration ../../resources/kieker.monitoring.properties`
- Start the Peass process: `cd peass && ./peass measure -executionFile ../zigbee_results/traceTestSelection_org.openhab.binding.zigbee.json -folder ../org.openhab.binding.zigbee -useKieker --measurementStrategy SEQUENTIAL -notUseAggregation -disableKiekerKoPeMe --record OPERATIONEXECUTION -iterations 10 -repetitions 10 -vms 2`
