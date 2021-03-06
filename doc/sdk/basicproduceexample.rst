Basic Produce Example
=====================

This sample demonstrates how to produce records to the DXL streaming service.

Prerequisites
*************

* A DXL streaming service is available for the sample to connect to.
* Credentials for the service available for use with the sample.

Setup
*****

Modify the example to include the appropriate settings for the streaming
service channel:

    .. code-block:: python

        CHANNEL_URL = "http://127.0.0.1:50080"
        CHANNEL_USERNAME = "me"
        CHANNEL_PASSWORD = "secret"
        CHANNEL_TOPIC = "my-topic"
        # Path to a CA bundle file containing certificates of trusted CAs. The CA
        # bundle is used to validate that the certificate of the server being connected
        # to was signed by a valid authority. If set to an empty string, the server
        # certificate is not validated.
        VERIFY_CERTIFICATE_BUNDLE = ""


For testing purposes, you can use the ``fake_streaming_service`` Python tool
embedded in the OpenDXL Streaming Client SDK to start up a local
streaming service. The initial settings in the example above include the URL
and credentials used by the ``fake_streaming_service``.

To launch the ``fake_streaming_service`` tool, run the following command in
a command window:

    .. code-block:: shell

        python sample/fake_streaming_service.py

Messages like the following should appear in the command window:

    .. code-block:: shell

        INFO:__main__:Starting service
        INFO:__main__:Started service on http://mycaseserver:50080

Running
*******

To run this sample execute the ``sample/basic/basic_produce_example.py`` script
as follows:

    .. parsed-literal::

        python sample/basic/basic_produce_example.py

If the records are successfully produced to the streaming service, the
following line should appear in the output window:

    .. parsed-literal::

        Succeeded.

To validate that the records were produced to the streaming service with
the expected content, you can execute the
``sample/basic/basic_consume_example.py`` script as follows:

    .. parsed-literal::

        python sample/basic/basic_consume_example.py

One of the records received by the sample should appear similar to the
following:

    .. code-block:: shell

        2018-05-30 17:35:36,754 __main__ - INFO - Received payloads:
        [
            ...
            {
                "message": "Hello from OpenDXL"
            }
            ...
        ]

Details
*******

The majority of the sample code is shown below:

    .. code-block:: python

        CHANNEL_TOPIC = "my-topic"

        # Create the message payload to be included in a record
        message_payload = {
            "message": "Hello from OpenDXL"
        }

        # Create the full payload with records to produce to the channel
        channel_payload = {
            "records": [
                {
                    "routingData": {
                        "topic": CHANNEL_TOPIC,
                        "shardingKey": ""
                    },
                    "message": {
                        "headers": {},
                        # Convert the message payload from a dictionary to a
                        # base64-encoded string.
                        "payload": base64.b64encode(
                            json.dumps(message_payload).encode()).decode()
                    }
                }
            ]
        }

        # Create a new channel object
        with Channel(CHANNEL_URL,
                     auth=ChannelAuth(CHANNEL_URL,
                                      CHANNEL_USERNAME,
                                      CHANNEL_PASSWORD,
                                      verify_cert_bundle=VERIFY_CERTIFICATE_BUNDLE),
                     verify_cert_bundle=VERIFY_CERTIFICATE_BUNDLE) as channel:
            # Produce the payload records to the channel
            channel.produce(channel_payload)

        print("Succeeded.")


The first step is to create a payload dictionary which includes an array of
records to be sent to the channel. The `message.payload` item in each record
is flattened from a dictionary into a string and encoded using the ``base64``
algorithm.

The next step is to create a :class:`dxlstreamingclient.channel.Channel`
instance, which establishes a channel to the streaming service. The channel
parameters include the URL to the streaming service, ``CHANNEL_URL``, and
credentials that the client uses to authenticate itself to the service,
``CHANNEL_USERNAME`` and ``CHANNEL_PASSWORD``.

The final step is to call the
:meth:`dxlstreamingclient.channel.Channel.produce` method with the payload of
records to be produced to the channel. Assuming the records can be produced
successfully, the text "Succeeded." should appear in the console output.
