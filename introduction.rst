
.. note::

  TBD: Add the text for the introduction here.
  This should contain the rationale for why we are doing this (Michael Reuter) etc.

This proposal focuses on two main topics:

- Improving the compatibility of system interfaces with InfluxDB.
- Replacing the XML interface definition format with native Apache Avro schemas.

Details regarding the InfluxDB compatibility issue can be found in :cite:`SQR-113`.
In essence, InfluxDB is designed around the concept of a "measurement" (a scalar data point associated with a timestamp), which does not map cleanly to array fields.

The Rubin Observatory Control System currently relies on arrays for two primary use cases: simultaneous multi-sensor measurements and packed time-series telemetry.

Simultaneous measurements occur when data is collected concurrently across multiple channels or sensors.
For example:

- **M1M3** measures force applied across 189 individual actuators simultaneously and packs these into a single "force" array topic.
- **ESS** measures temperatures across multiple physical sensors simultaneously and packs the values into a single "temperature" array topic.

Packed time series, on the other hand, allow a low-level controller to sample telemetry at high frequencies and publish the aggregated points in a single lower-frequency message, reducing transport overhead.
A prominent example is the **ATMCS**, where the low-level cRIO controller accumulates telemetry at 100 Hz and publishes an array of values once per second (1 Hz).

To resolve these issues, we propose deprecating array-based topic fields in favor of atomic, labeled data structures:

- For **simultaneous measurements** (e.g., M1M3 and ESS), topics containing arrays will be replaced by topics carrying a scalar values paired with an explicit identifier (e.g., `actuatorId` or `sensorId`).
- For **packed time-series telemetry**, the CSC will unpack individual data points and publish them as separate samples with their true, associated measurement timestamps (supported by updating `salobj` to accept explicit sample timestamps).

The second part of the proposal—replacing XML topic definitions with native Avro schemas—eliminates redundant architecture.
Following the migration from DDS to Kafka (:cite:`TSTN-028,TSTN-033`), Avro is already used internally by the schema registry for message serialization.
Currently, an intermediate translation stage converts XML topic definitions into Avro schemas.
Defining topics natively in Avro removes this bespoke translation layer and gives us direct access to advanced Avro schema features.
However, a lightweight preprocessing step will still be required to update the user-defined schemas with the standard fields and generate the generic topic definitions.

In the following sessions we will go over details about this proposal.
We start with a performance evaluation, where we measure the impact on client load and latency when going from large messages published at lower-frequency to smaller messages published at high-frequency.
We then discuss the convertion from xml to avro, reviewing a few details from the current interface and how they match Avro's specification.
Next we discuss a few use-cases of CSCs that we deem worth exploring, especially those that make heavy use of arrays.
Finally, we go over the expected timeline for the transition and some final words of conclusion.
