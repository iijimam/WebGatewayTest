FROM containers.intersystems.com/intersystems/iris:2022.2.0.368.0

USER ${ISC_PACKAGE_MGRUSER}
COPY  iris.script /tmp/iris.script

RUN iris start IRIS \
    && iris session IRIS < /tmp/iris.script \
    && iris stop IRIS quietly