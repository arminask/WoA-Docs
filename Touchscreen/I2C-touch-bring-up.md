# I2C touchscreen controller bring-up for Qualcomm Windows platforms

This documentation will show how touchscreen support for Windows is added.

All of the shown examples will be used from Galaxy A52s (a52sxq) SM7325 platform touch bring-up.

---

### Gathering information from the downstream Linux device tree

First, we need to find the touchscreen node in the device tree. To find that in dts, you can use the search for keywords like `touchscreen`, `touch`, etc.
