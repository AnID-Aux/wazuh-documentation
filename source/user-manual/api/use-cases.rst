.. Copyright (C) 2015, Wazuh, Inc.

.. meta::
   :description: This section presents use cases for the Wazuh API.
    
.. _wazuh_api_use_cases:

Use cases
---------

This section will present several use cases to give you a taste for the Wazuh API's potential. Details about all possible API requests can be found in the :ref:`reference <api_reference>` section.

Exploring the ruleset
^^^^^^^^^^^^^^^^^^^^^

Often when an alert fires, it is helpful to know details about the rule itself. The following request enumerates the attributes of rule *1002*:

.. code-block:: console

    # curl -k -X GET "https://localhost:55000/rules?rule_ids=1002&pretty=true" -H  "Authorization: Bearer $TOKEN"

.. code-block:: json
    :class: output

    {
       "data": {
          "affected_items": [
             {
                "filename": "0020-syslog_rules.xml",
                "relative_dirname": "ruleset/rules",
                "id": 1002,
                "level": 2,
                "status": "enabled",
                "details": {
                   "match": {
                      "pattern": "core_dumped|failure|error|attack| bad |illegal |denied|refused|unauthorized|fatal|failed|Segmentation Fault|Corrupted"
                    }
                },
                "pci_dss": [],
                "gpg13": [
                   "4.4"
                ],
                "gdpr": [],
                "hipaa": [],
                "nist_800_53": [],
                "groups": [
                   "syslog",
                   "errors"
                ],
                "description": "Unknown problem somewhere in the system."
             }
          ],
          "total_affected_items": 1,
          "total_failed_items": 0,
          "failed_items": []
       },
       "message": "All selected rules were returned",
       "error": 0
    }


It can also be helpful to know which rules matching a specific criteria are available. For example, all the rules with a group of **web**, a PCI tag of **10.6.1**, and containing the word **failures** can be showed using the command below:

.. code-block:: console

    # curl -k -X GET "https://localhost:55000/rules?pretty=true&limit=500&search=failures&group=web&pci_dss=10.6.1" -H  "Authorization: Bearer $TOKEN"

.. code-block:: json
    :class: output

    {
      "data": {
        "affected_items": [
          {
            "filename": "0260-nginx_rules.xml",
            "relative_dirname": "ruleset/rules",
            "id": 31316,
            "level": 10,
            "status": "enabled",
            "details": {
              "frequency": "8",
              "timeframe": "240",
              "if_matched_sid": "31315",
              "same_source_ip": "",
              "mitre": "\n      "
            },
            "pci_dss": [
              "10.6.1",
              "10.2.4",
              "10.2.5",
              "11.4"
            ],
            "gpg13": [
              "7.1"
            ],
            "gdpr": [
              "IV_35.7.d",
              "IV_32.2"
            ],
            "hipaa": [
              "164.312.b"
            ],
            "nist_800_53": [
              "AU.6",
              "AU.14",
              "AC.7",
              "SI.4"
            ],
            "groups": [
              "authentication_failures",
              "tsc_CC7.2",
              "tsc_CC7.3",
              "tsc_CC6.1",
              "tsc_CC6.8",
              "nginx",
              "web"
            ],
            "description": "Nginx: Multiple web authentication failures."
          }
        ],
        "total_affected_items": 1,
        "total_failed_items": 0,
        "failed_items": []
      },
      "message": "All selected rules were returned",
      "error": 0
    }



Testing rules and decoders
^^^^^^^^^^^^^^^^^^^^^^^^^^

With the Wazuh API, it is possible to start a **** session or use an already started session to test and verify custom or default rules and decoders. With the following request, a logtest session is created and the rules and decoders that match with the given log are shown. The predecoding phase is also shown, among other information.

.. code-block:: console

    # curl -k -X PUT "https://localhost:55000/logtest" -H  "Authorization: Bearer $TOKEN" -H  "Content-Type: application/json" -d "{\"event\":\"Jun 29 15:54:13 focal multipathd[557]: sdb: failed to get sysfs uid: No data available\",\"log_format\":\"syslog\",\"location\":\"user->/var/log/syslog\"}"


.. code-block:: json
    :class: output

    {
      "error": 0,
      "data": {
        "token": "bc3ca27a",
        "messages": [
          "WARNING: (7309): 'null' is not a valid token",
          "INFO: (7202): Session initialized with token 'bc3ca27a'"
        ],
        "output": {
          "timestamp": "2020-10-15T09:40:53.630+0000",
          "rule": {
            "level": 0,
            "description": "FreeIPA messages grouped",
            "id": "82202",
            "firedtimes": 1,
            "mail": false,
            "groups": [
              "freeipa"
            ]
          },
          "agent": {
            "id": "000",
            "name": "wazuh-master"
          },
          "manager": {
            "name": "wazuh-master"
          },
          "id": "1602754853.1000774",
          "cluster": {
            "name": "wazuh",
            "node": "master-node"
          },
          "full_log": "Jun 29 15:54:13 focal multipathd[557]: sdb: failed to get sysfs uid: No data available",
          "predecoder": {
            "program_name": "multipathd",
            "timestamp": "Jun 29 15:54:13",
            "hostname": "focal"
          },
          "decoder": {
            "name": "freeipa"
          },
          "location": "user->/var/log/syslog"
        },
        "alert": false,
        "codemsg": 1
      }
    }



Mining the file integrity monitoring database of an agent
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The API can be used to show information about all monitored files by syscheck. The following example shows all events related with *.py* files in agent *000* (the manager):

.. code-block:: console

    # curl -k -X GET "https://localhost:55000/syscheck/000?pretty=true&search=.py" -H  "Authorization: Bearer $TOKEN"

.. code-block:: json
    :class: output

    {
      "data": {
        "affected_items": [
          {
            "file": "/etc/python2.7/sitecustomize.py",
            "perm": "rw-r--r--",
            "sha1": "67b0a8ccf18bf5d2eb8c7f214b5a5d0d4a5e409d",
            "changes": 1,
            "md5": "d6b276695157bde06a56ba1b2bc53670",
            "inode": 29654607,
            "size": 155,
            "uid": "0",
            "gname": "root",
            "mtime": "2020-04-15T17:20:14Z",
            "sha256": "43d81125d92376b1a69d53a71126a041cc9a18d8080e92dea0a2ae23be138b1e",
            "date": "2020-05-25T14:28:41Z",
            "uname": "root",
            "type": "file",
            "gid": "0"
          },
          {
            "file": "/etc/python3.6/sitecustomize.py",
            "perm": "rw-r--r--",
            "sha1": "67b0a8ccf18bf5d2eb8c7f214b5a5d0d4a5e409d",
            "changes": 1,
            "md5": "d6b276695157bde06a56ba1b2bc53670",
            "inode": 29762235,
            "size": 155,
            "uid": "0",
            "gname": "root",
            "mtime": "2020-04-18T01:56:04Z",
            "sha256": "43d81125d92376b1a69d53a71126a041cc9a18d8080e92dea0a2ae23be138b1e",
            "date": "2020-05-25T14:28:41Z",
            "uname": "root",
            "type": "file",
            "gid": "0"
          }
        ],
        "total_affected_items": 2,
        "total_failed_items": 0,
        "failed_items": []
      },
      "message": "FIM findings of the agent were returned",
      "error": 0
    }

You can find a file using its md5/sha1 hash. In the following examples, the same file is retrieved using both its md5 and sha1:

.. code-block:: console

    # curl -k -X GET "https://localhost:55000/syscheck/000?pretty=true&hash=bc929cb047b79d5c16514f2c553e6b759abfb1b8" -H  "Authorization: Bearer $TOKEN"

.. code-block:: json
    :class: output

    {
      "data": {
        "affected_items": [
          {
            "file": "/sbin/swapon",
            "perm": "rwxr-xr-x",
            "sha1": "bc929cb047b79d5c16514f2c553e6b759abfb1b8",
            "changes": 1,
            "md5": "085c1161d814a8863562694b3819f6a5",
            "inode": 14025822,
            "size": 47184,
            "uid": "0",
            "gname": "root",
            "mtime": "2020-01-08T18:31:23Z",
            "sha256": "f274025a1e4870301c5678568ab9519152f49d3cb907c01f7c71ff17b1a6e870",
            "date": "2020-05-25T14:29:44Z",
            "uname": "root",
            "type": "file",
            "gid": "0"
          }
        ],
        "total_affected_items": 1,
        "total_failed_items": 0,
        "failed_items": []
      },
      "message": "FIM findings of the agent were returned",
      "error": 0
    }

.. code-block:: console

    # curl -k -X GET "https://localhost:55000/syscheck/000?pretty=true&hash=085c1161d814a8863562694b3819f6a5" -H  "Authorization: Bearer $TOKEN"

.. code-block:: json
    :class: output

    {
      "data": {
        "affected_items": [
          {
            "file": "/sbin/swapon",
            "perm": "rwxr-xr-x",
            "sha1": "bc929cb047b79d5c16514f2c553e6b759abfb1b8",
            "changes": 1,
            "md5": "085c1161d814a8863562694b3819f6a5",
            "inode": 14025822,
            "size": 47184,
            "uid": "0",
            "gname": "root",
            "mtime": "2020-01-08T18:31:23Z",
            "sha256": "f274025a1e4870301c5678568ab9519152f49d3cb907c01f7c71ff17b1a6e870",
            "date": "2020-05-25T14:29:44Z",
            "uname": "root",
            "type": "file",
            "gid": "0"
          }
        ],
        "total_affected_items": 1,
        "total_failed_items": 0,
        "failed_items": []
      },
      "message": "FIM findings of the agent were returned",
      "error": 0
    }

Getting information about the manager
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Some information about the manager can be retrieved using the Wazuh API. Configuration, status, information, logs, etc. The following example retrieves the status of each Wazuh daemon:

.. code-block:: console

    # curl -k -X GET "https://localhost:55000/manager/status?pretty=true" -H  "Authorization: Bearer $TOKEN"

.. code-block:: json
    :class: output

    {
      "data": {
        "affected_items": [
          {
            "wazuh-clusterd": "running",
            "wazuh-apid": "stopped"
          }
        ],
        "total_affected_items": 1,
        "total_failed_items": 0,
        "failed_items": []
      },
      "message": "Processes status were successfully read in specified node",
      "error": 0
    }


You can even dump the manager current configuration with the request below (response shortened for brevity):

.. code-block:: console

    # curl -k -X GET "https://localhost:55000/manager/configuration?pretty=true&section=global" -H  "Authorization: Bearer $TOKEN"

.. code-block:: json
    :class: output

    {
      "data": {
        "affected_items": [
          {
            "global": {
              "jsonout_output": "yes",
              "alerts_log": "yes",
              "logall": "no",
              "logall_json": "no",
              "email_notification": "yes",
              "email_to": "me@test.example",
              "smtp_server": "mail.test.example",
              "email_from": "wazuh@test.example",
              "email_maxperhour": "12",
              "email_log_source": "alerts.log",
              "white_list": [
                "127.0.0.1",
                "^localhost.localdomain$",
                "8.8.8.8",
                "8.8.4.4"
              ]
            }
          }
        ],
        "total_affected_items": 1,
        "total_failed_items": 0,
        "failed_items": []
      },
      "message": "Configuration was successfully read in specified node",
      "error": 0
    }


Playing with agents
^^^^^^^^^^^^^^^^^^^

Here are some commands for working with the agents.

This enumerates 2 **active** agents:

.. code-block:: console

    # curl -k -X GET "https://localhost:55000/agents?pretty=true&offset=1&limit=2&select=status%2Cid%2Cmanager%2Cname%2Cnode_name%2Cversion&status=active" -H  "Authorization: Bearer $TOKEN"

.. code-block:: json
    :class: output

    {
      "data": {
        "affected_items": [
          {
            "node_name": "worker2",
            "status": "active",
            "manager": "wazuh-worker2",
            "version": "Wazuh v3.13.1",
            "id": "001",
            "name": "wazuh-agent1"
          },
          {
            "node_name": "worker2",
            "status": "active",
            "manager": "wazuh-worker2",
            "version": "Wazuh v3.13.1",
            "id": "002",
            "name": "wazuh-agent2"
          }
        ],
        "total_affected_items": 9,
        "total_failed_items": 0,
        "failed_items": []
      },
      "message": "All selected agents information was returned",
      "error": 0
    }


Adding an agent is now easier than ever. Simply send a request with the agent name and its IP address.

.. code-block:: console

    # curl -k -X POST "https://localhost:55000/agents?pretty=true" -H  "Authorization: Bearer $TOKEN" -H  "Content-Type: application/json" -d "{\"name\":\"NewHost\",\"ip\":\"10.0.10.11\"}"

.. code-block:: json
    :class: output

    {
      "data": {
        "id": "013",
        "key": "MDEzIE5ld0hvc3RfMiAxMC4wLjEwLjEyIDkzOTE0MmE4OTQ4YTNlMzA0ZTdiYzVmZTRhN2Q4Y2I1MjgwMWIxNDI4NWMzMzk3N2U5MWU5NGJiMDc4ZDEzNjc="
      },
      "error": 0
    }

Ingest security events
^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 4.6.0

You can send security events for analysis using the Wazuh API.

There's a limit of ``30`` requests per minute and 100 events per request. This limit prevents endpoints to ingest large amounts of data too fast. Check :ref:`max_request_per_minute <api_configuration_access>` to lower this limit even further or disable the feature.

.. code-block:: console

    # curl -k -X POST "https://localhost:55000/events" -H  "Authorization: Bearer $TOKEN" -H  "Content-Type: application/json" -d '{"events": ["Event value 1", "{\"someKey\": \"Event value 2\"}"]}'

.. code-block:: json
    :class: output

    {
      "data": {
        "affected_items": [

        ],
        "total_affected_items": 2,
        "total_failed_items": 0,
        "failed_items": []
      },
      "message": "All events were forwarded to analisysd",
      "error": 0
    }

Conclusion
^^^^^^^^^^
The provided examples should help appreciate the potential of the Wazuh API. Remember to check out the :ref:`reference <api_reference>` document to discover all the available API requests.
