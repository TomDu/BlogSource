---
title: Create Docker Image for SSIS on Linux
date: 2018-07-28 19:00:58
tags:
- SSIS on Linux
- Docker
---

# Resources

- [Extract, transform, and load data on Linux with SSIS](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-migrate-ssis)
- [Install SQL Server Integration Services (SSIS) on Linux](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-setup-ssis)
- [Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/)

<!-- more -->

# Step by step

Wrapper for `dtexec` under `/opt/ssis/bin`:

```bash
#!/bin/bash

while [[ $# -gt 0 ]]
do
    key="$1"

    case $key in
    /f)
    PACKAGE="$2"
    shift # past argument
    ;;
    /de)
    PASSWORD="$2"
    shift
    ;;
    /out)
    RESULT_FILE="$2"
    shift
    ;;
    *)
    # Unknown option, bail out of arg parsing
    # so we can pass it to the windows process.
    break
    ;;
    esac

    shift # past argument or value
done

SECONDS=0

CMD="/opt/ssis/bin/dtexec /f $PACKAGE /de $PASSWORD"

EXECUTION_OUTPUT=$($CMD 2>&1)

RESULT=`/bin/echo "$EXECUTION_OUTPUT" | /bin/grep -F "DTExec: The package execution returned" | /usr/bin/head -1`
read res <<<${RESULT//[^0-9]/ }
if [[ $res == 0 ]] ; then
    RESULT="SUCCESS"
else
    RESULT="FAIL"
    echo "$EXECUTION_OUTPUT"
fi

OUTPUT="$RESULT $SECONDS"
echo $OUTPUT | tee $RESULT_FILE

#/usr/bin/printf "Duration: "%dh:%dm:%ds"\n" $(($SECONDS/3600)) $(($SECONDS%3600/60)) $(($SECONDS%60))
```

Dockerfile:

```Dockerfile
FROM ubuntu

COPY mssql-server-is_xxx_amd64.deb .

# This is to solve the issue:
#  The following packages have unmet dependencies:
#  mssql-server : Depends: openssl (<= 1.1.0).
# see https://askubuntu.com/questions/930712/installation-problems-with-ms-sql-server-for-linux?stw=2#1033154
COPY openssl_1.0.2g-1ubuntu4_amd64.deb .

RUN apt update \
    && apt install -y ./openssl_1.0.2g-1ubuntu4_amd64.deb \
    && apt install -y ./mssql-server-is_xxx_amd64.deb \
    && mkdir -p /var/opt/ssis

# Content in ssis.conf:
#  [TELEMETRY]
#  enabled = N
# This is to avoid errors when the script calls another process to do the licensing
COPY ssis.conf /var/opt/ssis/

RUN SSIS_PID=Enterprise ACCEPT_EULA=Y /opt/ssis/bin/ssis-conf -n setup \
    && rm -rf mssql-server-is_14.0.3015.40-1_amd64.deb \
    && rm -rf openssl_1.0.2g-1ubuntu4_amd64.deb

# The wrapper
COPY dtexec .

#RUN apt update \
#    && apt install -y libunwind8-dev \
#    && apt install -y libnuma1 \
#    && apt install -y libjemalloc1 \
#    && apt install -y libc++1 \
#    && apt install -y python2.7 python-pip \
#    && apt install ./mssql-server-is_xxx_amd64.deb \
#    && SSIS_PID=Enterprise ACCEPT_EULA=Y /opt/ssis/bin/ssis-conf -n setup \
#    && rm -rf mssql-server-is_14.0.3015.40-1_amd64.deb

ENTRYPOINT ["./dtexec"]
```

Build Docker image under the folder where Dockerfile is placed:

```bash
docker build -t ssis-docker .
```

Run the image:

```bash
docker run -v $(pwd):/root ssis-docker /f /root/test.dtsx /de test /out /root/out.txt
```

I've uploaded the image to [tomdu/ssis-docker](https://hub.docker.com/r/tomdu/ssis-docker/).

~END~
