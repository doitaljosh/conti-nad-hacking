# conti-nad-hacking
Reverse engineering a Continental telematics control unit

Note: this method only works with Continental DCMs that have a USB connection, otherwise, one will need to be populated by soldering it.
The one used here is a TCU from a 2018 Subaru Forester containing a Continental LNAD modem module.

1. Add the appropriate device entries to `option.c` and `qmi_wwan.c`
2. Issue `make modules SUBDIRS=drivers/usb/serial` and `make modules SUBDIRS=drivers/net/usb/` to re-compile USB drivers.
3. Issue `make modules_install INSTALL_MOD_PATH=/` to install the new modules. In some cases, you may need to re-compile the entire kernel if one of these drivers or it's dependencies is a built-in module.
4. Connect a USB cable from the host to the DCM's brown USB connector. The pinout from top left to bottom right is GND, DM, DP, VBUS.
5. Considering that the drivers were correctly patched and compiled, you should see 5 new serial ports in the form of `/dev/ttyUSB*`

### option.c
```

// Add right before device flags
/* Continental Automotive Systems */
#define CAS_VENDOR_ID				0x1e3a
#define CAS_PRODUCT_STARLINK			0x0002

// Add to bottom of option_ids struct
{ USB_DEVICE(CAS_VENDOR_ID, CAS_PRODUCT_STARLINK) }, 			/* Subaru Starlink Telematics */
```

### qmi_wwan.c
```
// Add to bottom of cdc_devs struct
// Continental Automotive Systems LNAD
{ USB_DEVICE_AND_INTERFACE_INFO(0x1e3a, 0x0002,
		USB_CLASS_COMM,
		USB_CDC_SUBCLASS_NCM, USB_CDC_PROTO_NONE),
		.driver_info = (unsigned long)&wwan_info,
},
```
