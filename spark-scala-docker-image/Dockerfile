FROM --platform=linux/amd64 ghcr.io/kubeflow/kubeflow/notebook-servers/jupyter-scipy:v1.10.0

USER root

# Install Java 17
RUN apt-get update && apt-get install -y \
    openjdk-17-jdk \
    curl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
ENV PATH=$JAVA_HOME/bin:$PATH

# NOTE: JAVA_TOOL_OPTIONS is deliberately NOT set at image level.
# A container-wide value is inherited by Jupyter's own JVM subprocesses and
# breaks the notebook UI (server serves HTML but the front-end never initializes).
# The Spark module options are injected into the kernel spec instead (below),
# so they apply only to the Scala kernel's JVM.

# Install Coursier
RUN curl -fL https://github.com/coursier/coursier/releases/latest/download/cs-x86_64-pc-linux.gz | \
    gzip -d > /usr/local/bin/cs && \
    chmod +x /usr/local/bin/cs

# Install Almond Scala 2.13 kernel
RUN cs launch --use-bootstrap almond:0.14.1 --scala 2.13.16 -- \
    --install \
    --global \
    --id scala213 \
    --display-name "Scala 2.13 (Spark 4.0.1)"

# Inject the full Spark 4 / Java 17 module option set into the KERNEL spec only,
# via a JAVA_TOOL_OPTIONS env scoped to this kernel (not the whole container).
RUN python3 - <<'PYEOF'
import json, pathlib
kj = pathlib.Path("/usr/local/share/jupyter/kernels/scala213/kernel.json")
spec = json.loads(kj.read_text())
opts = " ".join([
    "-XX:+IgnoreUnrecognizedVMOptions",
    "-Xmx4g",
    "--add-exports=java.base/java.lang=ALL-UNNAMED",
    "--add-exports=java.base/java.lang.invoke=ALL-UNNAMED",
    "--add-exports=java.base/java.lang.reflect=ALL-UNNAMED",
    "--add-exports=java.base/java.io=ALL-UNNAMED",
    "--add-exports=java.base/java.net=ALL-UNNAMED",
    "--add-exports=java.base/java.nio=ALL-UNNAMED",
    "--add-exports=java.base/java.util=ALL-UNNAMED",
    "--add-exports=java.base/java.util.concurrent=ALL-UNNAMED",
    "--add-exports=java.base/java.util.concurrent.atomic=ALL-UNNAMED",
    "--add-exports=java.base/sun.nio.ch=ALL-UNNAMED",
    "--add-exports=java.base/sun.nio.cs=ALL-UNNAMED",
    "--add-exports=java.base/sun.security.action=ALL-UNNAMED",
    "--add-exports=java.base/sun.util.calendar=ALL-UNNAMED",
    "--add-exports=java.security.jgss/sun.security.krb5=ALL-UNNAMED",
    "--add-opens=java.base/java.lang=ALL-UNNAMED",
    "--add-opens=java.base/java.lang.invoke=ALL-UNNAMED",
    "--add-opens=java.base/java.lang.reflect=ALL-UNNAMED",
    "--add-opens=java.base/java.io=ALL-UNNAMED",
    "--add-opens=java.base/java.net=ALL-UNNAMED",
    "--add-opens=java.base/java.nio=ALL-UNNAMED",
    "--add-opens=java.base/java.util=ALL-UNNAMED",
    "--add-opens=java.base/java.util.concurrent=ALL-UNNAMED",
    "--add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED",
    "--add-opens=java.base/jdk.internal.ref=ALL-UNNAMED",
    "--add-opens=java.base/sun.nio.ch=ALL-UNNAMED",
    "--add-opens=java.base/sun.nio.cs=ALL-UNNAMED",
    "--add-opens=java.base/sun.security.action=ALL-UNNAMED",
    "--add-opens=java.base/sun.util.calendar=ALL-UNNAMED",
    "--add-opens=java.security.jgss/sun.security.krb5=ALL-UNNAMED",
    "-Djdk.reflect.useDirectMethodHandle=false",
    "-Dio.netty.tryReflectionSetAccessible=true",
])
spec.setdefault("env", {})["JAVA_TOOL_OPTIONS"] = opts
kj.write_text(json.dumps(spec, indent=1))
print("patched kernel.json env with", len(opts.split()), "java opts")
PYEOF

# Make kernel launcher readable by jovyan (installed as root)
RUN chmod -R a+rX /usr/local/share/jupyter/kernels

# Install PySpark + Spark Connect deps
RUN pip install --no-cache-dir \
    pyspark==4.0.1 \
    grpcio \
    grpcio-status \
    googleapis-common-protos \
    pyarrow

USER $NB_UID
