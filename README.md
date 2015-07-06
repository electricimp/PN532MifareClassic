# PN532 MIFARE Classic Library

This library exposes access to the MIFARE classic functionality of the [NXP PN532/C106 NFC Module](http://www.nxp.com/documents/short_data_sheet/PN532_SDS.pdf).  It provides functions needed to read and write to [MIFARE Classic](http://www.nxp.com/products/identification_and_security/smart_card_ics/mifare_smart_card_ics/mifare_classic/) tags, which support read/write storage.  It also serves as an example of how to build on the [PN532 library](https://github.com/electricimp/PN532) to interface with the many other protocols and features that the PN532 supports.

Note that this library depends on the [PN532 library](https://github.com/electricimp/PN532).

### Examples

For an example of using the `PN532MifareClassic` class to build an ID card reader, see the [MIFARE reader example](examples/reader).

## Constructor: PN532MifareClassic(*pn532*)

Associates this class with a previously constructed [PN532](https://github.com/electricimp/PN532) object.

The *pn532* object must have been constructed with a version of the `PN532` library with a version number in the 1.x.x major revision of at least 1.0.0.

### Usage

```squirrel
#require "PN532.class.nut:1.0.0"
#require "PN532MifareClassic.class.nut:1.0.0"

// ... setup PN532 pins and callback ...

local reader = PN532(spi, nss, rstpd_l, irq, constructorCallback);
mifareReader <- PN532MifareClassic(reader);
```

## pollNearbyTags(*pollAttempts, pollPeriod, callback*)

A wrapper around the PN532 class's [pollNearbyTags(*tagType, pollAttempts, pollPeriod, callback*)](https://github.com/electricimp/PN532#pollnearbytagstagtype-pollattempts-pollperiod-callback) method that configures it to search for MIFARE NFC tags.

### Usage

```squirrel
mifareReader.pollNearbyTags(0x0A, 6, scanCallback);
```

## authenticate(*tagSerial, address, aOrB, key, callback*)

Attempts to authenticate the reader to perform operations on a specified EEPROM address.

### Parameters

- *tagSerial*: A blob representing the UID of the tag to be authenticated. This UID can be taken from the response to the [pollNearbyTags(*pollAttempts, pollPeriod, callback*)](#pollnearbytagspollattempts-pollperiod-callback) call.
- *address*: The address of the memory block to be authenticated. For the MIFARE Classic 1k, this is an integer in the range 0-63.
- *aOrB*: The key type to authenticate against.  The options are the static class members `PN532MifareClassic.AUTH_TYPE_A` and `PN532MifareClassic.AUTH_TYPE_B`.
- *key*: The key to use when authenticating for the given block.  If no key has been set, set this parameter to null to use the default FFFFFFFFFFFFh value.
- *callback*: A function taking the following arguments:
    - *error*: A string that is null on success.
    - *status*: A boolean that is set to true if the authentication succeeded.

### Usage

```squirrel
function authCallback(error, status) {
    if(error != null) {
        server.log(error);
        return;
    }
    
    if(authenticated) {
        // Operate on tag...
    }
}

mifareReader.authenticate(tagData.NFCID, 0x2, PN532MifareClassic.AUTH_TYPE_A, null, authCallback);
```

## read(*address, callback*)

Reads 16 bytes from the *address* specified on the currently authenticated tag.

*callback*: A function taking the following arguments:

- *error*: A string that is null on success.
- *data*: A 16-byte blob containing the data read from the tag.

Note that this call will fail with an error if the address being read has not previously been authenticated.

### Usage

```squirrel
// ...authenticate the address...

function readCallback(error, data) {
    if(error != null) {
        server.log(error);
        return;
    }
    server.log(data.tostring());
}

mifareReader.read(0x2, readCallback);
```

## write(*address, data, callback*)

Writes 16 bytes of blob *data* to the *address* specified on the currently authenticated tag.

*callback* is a function taking an *error* string that is null on success.

### Errors
- If the address being read has not previously been authenticated, the callback will return a `PN532MifareClassic.ERROR_BAD_AUTHENTICATION` error.
- If *data* is not exactly 16 bytes long, the callback will return a `PN532MifareClassic.ERROR_INCORRECT_LENGTH` error. 

### Usage

```squirrel
// ...authenticate the address...

function writeCallback(error) {
    if(error != null) {
        server.log(error);
        return;
    }
}

local data = blob(16);
for(local i = 0; i < 16; i++) {
    data[i] = 88;
}

mifareReader.write(0x2, data, writeCallback);
```

# License

The PN532 MIFARE Classic library is licensed under the [MIT License](./LICENSE).
