# base image of ubuntu:18.04 due to environment with older machines 
# and easier to just have a single supported binary.
ARG TARGETARCH

FROM ${TARGETARCH}/ubuntu:18.04 AS builder
ARG TARGETARCH
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y \
    python3.8 python3.8-venv python3-pip libpython3.8-dev \
    build-essential \
    && rm -rf /var/lib/apt/lists/*
RUN python3.8 -m pip install --no-cache-dir --upgrade pip \
    && python3.8 -m pip install --no-cache-dir pyinstaller cryptography==3.4.8 cffi 

COPY . netbox-agent/

RUN python3.8 -m pip install -r netbox-agent/requirements.txt
RUN pip install ./netbox-agent/ \
&& printf '%s\n' \
  'from netbox_agent.cli import main' \
  '' \
  'if __name__ == "__main__":' \
  '    main()' \
  > netbox_agent_entry.py  && \
NETBOX_AGENT__NETBOX__URL=http://dummy     NETBOX_AGENT__NETBOX__TOKEN=dummy \
    python3.8 - <<'EOF'
from netbox_agent.network import Network
import inspect

src = inspect.getsource(Network.update_interface_macs)
print(src)
assert "Skipping MAC address object sync for virtual machine" in src
EOF

RUN pyinstaller \
  --onefile \
  --collect-submodules netbox_agent.drivers \
  --name netbox-agent-linux-${TARGETARCH} \
  netbox_agent_entry.py && \
  pip freeze > /dist/pip.${TARGETARCH}.freeze.txt

FROM scratch
COPY --from=builder /dist /dist/
