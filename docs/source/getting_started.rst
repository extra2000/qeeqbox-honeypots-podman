Getting Started
===============

Example how to use Qeeqbox Honeypots.

Build Qeeqbox Honeypots base image
----------------------------------

``cd`` into ``src/qeeqbox-honeypots`` and then execute the following command:

.. code-block:: bash

    cd src/qeeqbox-honeypots
    podman build -t qeeqbox/honeypots -f Dockerfile .

Build Qeeqbox Honeypots general image
-------------------------------------

Create honeypot exec script:

.. code-block:: bash

    cp -v service/run.sh{.example,}

``cd`` into ``src/qeeqbox-honeypots`` and then execute the following command:

.. code-block:: bash

    cd deployment/general
    podman build -t extra2000/qeeqbox-honeypots-general -f Dockerfile .

Create Podman Network (the ``qeeqboxnet``)
------------------------------------------

Create ``~/.config/cni/net.d/qeeqboxnet.conflist`` file:

.. code-block:: yaml

    {
      "cniVersion": "0.4.0",
      "name": "qeeqboxnet",
      "plugins": [
        {
          "type": "bridge",
          "bridge": "cni-podman1",
          "isGateway": true,
          "ipMasq": true,
          "hairpinMode": true,
          "ipam": {
            "type": "host-local",
            "routes": [{ "dst": "0.0.0.0/0" }],
            "ranges": [
              [
                {
                  "subnet": "192.168.125.0/24",
                  "gateway": "192.168.125.1"
                }
              ]
            ]
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        },
        {
          "type": "firewall"
        },
        {
          "type": "tuning"
        },
        {
          "type": "dnsname",
          "domainName": "qeeqboxnet"
        }
      ]
    }

.. note::

    If ``~/.config/cni/net.d/`` is not exists, create the directory using ``sudo mkdir -pv ~/.config/cni/net.d/``.

.. warning::

    Rename ``cni-podman1`` to ``cni-podman2`` and etc if the name is already used by other Podman deployments.

How to deploy
-------------

``cd`` into ``deployment/general``:

.. code-block:: bash

    cd deployment/general

Then, refer to the following Subsections below.

Create config files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/qeeqbox-honeypots-general.yaml{.example,}
    cp -v configs/config.json{.example,}

Create pod file
~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v qeeqbox-honeypots-general.yaml{.example,}

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/qeeqbox_honeypots_general.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "qeeqbox_honeypots_general"

Deployment
----------

Deploy ``qeeqbox-honeypots-general``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``cd`` into ``deployment/general``:

.. code-block:: bash

    cd deployment/general

For SELinux platform, label the following files to allow to be mounted into container:

.. code-block:: bash

    chcon -v -t container_file_t ./configs/config.json

Execute the following command:

.. code-block:: bash

    podman play kube --network qeeqboxnet --configmap configmaps/qeeqbox-honeypots-general.yaml --seccomp-profile-root ./seccomp qeeqbox-honeypots-general.yaml

Testing
-------

Test MySQL connection:

.. code-block:: bash

    podman run -it --rm --network=qeeqboxnet docker.io/library/mariadb:10.3 mysql -utest -ptest --host qeeqbox-honeypots-general-pod.qeeqboxnet --port 3306
