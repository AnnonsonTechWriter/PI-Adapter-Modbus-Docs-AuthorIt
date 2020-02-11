---
uid: OSIsoftAdapterForModbusTCPDataSelectionConfiguration
---

# OSIsoft Adapter for Modbus TCP data selection configuration

Once a data source is configured for a Modbus TCP instance, you must configure which data is to be collected from the designated source  device.

## Configure Modbus TCP data selection

Complete the following procedure to configure Modbus TCP data selection:

1. Using any text editor, create a file that contains a Modbus TCP data selection in JSON form.
    - For content structure, see [Modbus TCP data selection examples](#modbus-tcp-data-selection-examples). 
    - For a table of all available parameters, see [Modbus TCP data selection parameters](#modbus-tcp-data-selection-parameters). 
3. Save the file, for example as _DataSelection.config.json_.
4. Use any of the [Configuration tools](xref:ConfigurationTools) capable of making HTTP requests to execute a POST command with the contents of that file to the following endpoint: `http://localhost:5590/api/v1/configuration/<adapterId>/DataSelection/`.

    Example using curl (run this command from the same directory where the file is located):

    ```bash
    curl -d "@DataSelection.config.json" -H "Content-Type: application/json" "http://localhost:5590/api/v1/configuration/<adapterId>/DataSelection"
    ```

## Modbus TCP data selection schema

The full schema definition for the Modbus data selection configuration is in the _Modbus_DataSelection_schema.json_ located here:

Windows: *%Program Files%\OSIsoft\Adapters\Modbus\Schemas*

Linux: */opt/OSIsoft/Adapters/Modbus/Schemas*

## Modbus TCP data selection parameters

The following parameters are available for configuring a Modbus TCP data selection.

| Parameter | Required | Type | Nullable | Description |
|-----------|----------|------|----------|-------------|
| **Id** | Optional | `string` | Yes | This field is used to update an existing measurement. The ID automatically updates when there are changes to the measurement and will follow the format of `<UnitId`>.`<RegisterType`>.`<RegisterOffset`>.
| **Selected** | Optional | `Boolean` | No | This field is used to select or clear a measurement. To select an item, set to true. To remove an item, leave the field empty or set to false.  If not configured, the default value is true.|
| **Name** | Optional | `string` | Yes | The optional friendly name of the data item collected from the data source. If not configured, the default value will be the stream ID. |
| **UnitId** | Required | number | No | Modbus TCP slave device unit ID. This must be a value between 0 and 247, inclusively. |
| **RegisterType** | Required | number or `string` | No | Modbus TCP register type. Supported types are Coil, Discrete, Input16, Input32, Holding16 and Holding32.<br><br>Input16 and Holding16 are used to read registers that have a size of 16 bits. For registers that have a size of 32 bits, use the Input32 and Holding32 register types. To represent the types, you can type in the register type ID or the exact name: <br><br>1 or Coil (Read Coil Status)<br>2 or Discrete (Read Discrete Input Status)<br>3 or Holding16 (Read 16-bit Holding Registers)<br>4 or Holding32 (Read 32-bit Holding Registers)<br>6 or Input16 (Read 16-bit Input Registers)<br>7 or Input32 (Read 32-bit Input Registers) |
| **RegisterOffset** | Required | number | No | The 0 relative offset to the starting register for this measurement. For example, if your Holding registers start at base register 40001, the offset to this register is 0. For 40002, the offset to this register is 1.|
| **DataTypeCode** | Required | number | No | An integer representing the data type that Modbus TCP adapter will read starting at the register specified by the offset. Supported data types are:<br>1 = Boolean<br>10 = Int16<br>20 = UInt16<br>30 = Int32<br>31 = Int32ByteSwap<br>100 = Float32<br>101 = Float32ByteSwap<br>110 = Float64<br>111 = Float64ByteSwap<br>1001 - 1250 = String <br>2001 - 2250 = StringByteSwap |
| **ScanRate** | Required | number | No | How often this measurement should be read from the device in milliseconds. Acceptable values are from 0 to 86400000. If 0 ms is specified, Modbus TCP adapter will scan for data as fast as possible.|
| **BitMap** | Required | `string` | Yes | The bitmap is used to extract and reorder bits from a word register. The format of the bitmap is uuvvwwxxyyzz, where uu, vv, ww, yy, and zz each refer to a single bit. A leading zero is required if the referenced bit is less than 10. The low-order bit is 01 and high-order bit is either 16 or 32. Up to 16 bits can be referenced for a 16-bit word (data types 10 and 20) and up to 32 bits can be referenced for a 32-bit word (data type 30 and 31). The bitmap 0307120802 will map the second bit of the original word to the first bit of the new word, the eighth bit to the second bit, the twelfth bit to the third bit, and so on. The high-order bits of the new word are padded with zeros if they are not specified. |
| **ConversionFactor** | Required | number | Yes | This numerical value can be used to scale the raw response received from the Modbus TCP device. If this is specified, regardless of the specified data type, the value will be promoted to a float32 (single) when stored. [Result = (Value / Conversion Factor)] |
| **ConversionOffset** | Required | number | Yes | This numerical value can be used to apply an offset to the response received from the Modbus TCP device. If this is specified, regardless of the specified data type, the value will be promoted to a float32 (single) when stored.  [Result = (Value - Conversion Offset)] |
| **StreamID** | Required | `string` | Yes | The custom stream ID that will be used to create the streams. If not specified, the Modbus TCP adapter will generate a default stream ID based on the measurement configuration. A properly configured custom stream ID follows these rules:<br><br>Is not case-sensitive.<br>Can contain spaces.<br>Cannot start with two underscores ("__").<br>Can contain a maximum of 100 characters.<br>Cannot use the following characters: / : ? # [ ] @ ! $ & ' ( ) \ * + , ; = % < > &#124;<br>Cannot start or end with a period.<br>Cannot contain consecutive periods.<br>Cannot consist of only periods.

Each JSON object in the file represents a measurement. You can modify the fields in each object to configure the measurement parameters. To add more measurements, you need to create more JSON objects with properly completed fields.

## Modbus TCP data selection examples

The following are examples of valid Modbus TCP data selection configurations.

**Minimum data selection configuration:**

```json
[
    {
        "UnitId": 1,
        "RegisterType": 3,
        "RegisterOffset": 122,
        "DataTypeCode": 20,
        "ScanRate": 1000,
    }
]
```

**Maximum data selection configuration:**

```json
[
    {
        "Id": "DataItem1",
        "Selected": true,
        "Name": "MyDataItem",
        "UnitId": 1,
        "RegisterType": 3,
        "RegisterOffset": 123,
        "DataTypeCode": 20,
        "ScanRate": 300,
        "StreamId": "stream.1",
        "BitMap": "020301",
        "ConversionFactor": 12.3,
        "ConversionOffset": 14.5
    }
]
