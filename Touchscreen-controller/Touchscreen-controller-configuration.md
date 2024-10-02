# Bringing up touchscreen controller for Windows platform

This article will explain how to implement I2C touchscreen controller on Windows operating system using a downstream Linux device tree as reference.

All examples shown below were used from the implementation of an Android device with Qualcomm Snapdragon 778G SoC (SM7325).

---

### Downstream Linux device tree

Touchscreen node in the device tree. It is possible that there can be multiple touchscreen nodes which can be disregarded by checking if it's statuses are disabled.

The parent I2C node which the touchscreen uses will be the touchscreen controller's child block.

```c
		i2c@a94000 {
			compatible = "qcom,i2c-geni";
			reg = <0xa94000 0x4000>;
			#address-cells = <0x01>;
			#size-cells = <0x00>;
			interrupts = <0x00 0x166 0x04>;
			clock-names = "se-clk\0m-ahb\0s-ahb";
			clocks = <0x28 0x60 0x28 0x68 0x28 0x69>;
			pinctrl-names = "default\0sleep";
			pinctrl-0 = <0x1fb>;
			pinctrl-1 = <0x1fc>;
			dmas = <0x1e6 0x00 0x05 0x03 0x40 0x02 0x1e6 0x01 0x05 0x03 0x40 0x02>;
			dma-names = "tx\0rx";
			qcom,wrapper-core = <0x1e7>;
			status = "ok";
			phandle = <0x496>;
			qcom,i2c-touch-active = "novatek,NVT-ts";
			samsung,reset-before-trans;
			samsung,stop-after-trans;

			novatek@62 {
				compatible = "novatek,NVT-ts";
				reg = <0x62>;
				interrupt-parent = <0x3c>;
				interrupts = <0x51 0x2008>;
				pinctrl-names = "pmx_ts_active\0pmx_ts_suspend\0pmx_ts_release";
				pinctrl-0 = <0x454>;
				pinctrl-1 = <0x456 0x455>;
				pinctrl-2 = <0x457>;
				novatek,reset-gpio = <0x3c 0x69 0x00>;
				novatek,irq-gpio = <0x3c 0x51 0x2008>;
				panel = <0x5b1 0x5b2 0x5b3>;
				novatek,trusted-touch-mode = "vm_mode";
				novatek,touch-environment = "pvm";
				novatek,trusted-touch-spi-irq = <0x259>;
				novatek,trusted-touch-io-bases = <0xf134000 0xf135000 0xf169000 0xf151000 0xa94000 0xa10000>;
				novatek,trusted-touch-io-sizes = <0x1000 0x1000 0x1000 0x1000 0x1000 0x4000>;
				status = "disabled";
			};

			/* touchscreen@49 node is the active one, since novatek@62 is disabled */

			touchscreen@49 {
				status = "ok";
				compatible = "stm,stm_ts";
				reg = <0x49>;
				pinctrl-names = "on_state\0off_state";
				pinctrl-0 = <0x5b8>;
				pinctrl-1 = <0x5b9>;
				tsp_avdd_ldo-supply = <0x303>;
				sec,irq_gpio = <0x3c 0x51 0x00>;
				sec,project_name = "a52sxq\0a52sxq";
				sec,bringup = <0x00>;
				sec,ss_touch_num = <0x01>;
				sec,tclm_level = <0x02>;
				sec,afe_base = <0x27>;
				sec,area-size = <0x85 0x10a 0x155>;
				sec,max_coords = <0x1000 0x1000>;
				enable_settings_aot;
				support_dex_mode;
				support_fod;
				support_fod_lp_mode;
				support_mis_calibration_test;
				support_ear_detect_mode;
				support_open_short_test;
				sec,firmware_name = "tsp_stm/fts5cu56a_a52sxq.bin";
				sec,regulator_boot_on;
				sec,unuse_dvdd_power;
				phandle = <0x657>;
			};
		};
```
<br>

Relevant parts for the ACPI custom PEP0 LPXC packages modification.

The `touchscreen@49` node, line: `tsp_avdd_ldo-supply = <0x303>;` mentions an LDO regulator which can be found by it's phandle `<0x303>`:

```c
			rpmh-regulator-ldoc7 {
				compatible = "qcom,rpmh-vrm-regulator";
				qcom,resource-name = "ldoc7";
				qcom,regulator-type = "pmic5-ldo";
				qcom,supported-modes = <0x02 0x04>;
				qcom,mode-threshold-currents = <0x00 0x7530>;

				regulator-pm8350c-l7 {
					regulator-name = "pm8350c_l7";
					qcom,set = <0x03>;
					regulator-min-microvolt = <0x2dc6c0>;
					regulator-max-microvolt = <0x2dc6c0>;
					qcom,init-voltage = <0x2dc6c0>;
					qcom,init-mode = <0x04>;
					phandle = <0x303>;
					regulator-boot-on;
				};
			};
```
<br>

To know the correct LPMX packages order and their configuration in the PEP scope device, let's look at the `<0x5b8>` phandle which is mentioned in the touchscreen@49 node, line: `pinctrl-0 = <0x5b8>;`.

```
			stm_attn_irq {
				phandle = <0x5b8>;

				mux {
					pins = "gpio81";
					function = "gpio";
				};

				config {
					pins = "gpio81";
					input-enable;
					bias-disable;
				};
			};
```
<br>

The touchscreen@49 node has another `pinctrl-1 = <0x5b9>;` line with phandle `<0x5b9>`.

```c
			stm_attn_input {
				phandle = <0x5b9>;

				mux {
					pins = "gpio81";
					function = "gpio";
				};

				config {
					pins = "gpio81";
					input-enable;
					bias-pull-down;
				};
			};
```
<br>

### ACPI configuration

`TSC1` device configuration:

```c
        Device(TSC1)
        {
            Name(_HID, "STFT556A")
            Alias(\_SB_.PSUB, _SUB)
            Name(_DEP, Package(0x3)
            {
                \_SB_.GIO0,
                \_SB_.IC14,
                \_SB_.PEP0
            })
            Method(_CRS, 0x0, NotSerialized)
            {
                Name (RBUF, ResourceTemplate ()
                {
                    I2cSerialBusV2 (0x0049, ControllerInitiated, 0x00061A80,
                        AddressingMode7Bit, "\\_SB.IC14",
                        0x00, ResourceConsumer, , Exclusive,
                        )
                    GpioInt (Level, ActiveLow, Exclusive, PullUp, 0x0000,
                        "\\_SB.GIO0", 0x00, ResourceConsumer, ,
                        )
                        {   // Pin list
                            0x0051 // From touchscreen node, sec,irq_gpio = <0x3c 0x51 0x00>;
                        }
                })
                Return (RBUF) /* \_SB_.TSC1._CRS.RBUF */
            }
            Name(PGID, Buffer(0xa)
            {
    0x5c, 0x5f, 0x53, 0x42, 0x2e, 0x54, 0x53, 0x43, 0x31, 0x00
            })
            Name(DBUF, Buffer(DBFL)
            {
            })
            CreateByteField(DBUF, Zero, STAT)
            CreateByteField(DBUF, 0x2, DVAL)
            CreateField(DBUF, 0x18, 0xa0, DEID)
            Method(_S1D, 0x0, NotSerialized)
            {
                Return(0x3)
            }
            Method(_S2D, 0x0, NotSerialized)
            {
                Return(0x3)
            }
            Method(_S3D, 0x0, NotSerialized)
            {
                Return(0x3)
            }
            Method(_PS0, 0x0, NotSerialized)
            {
                Store(Buffer(ESNL)
                {
                }, DEID)
                Store(Zero, DVAL)
                Store(PGID, DEID)
                If(\_SB_.ABD_.AVBL)
                {
                    Store(DBUF, \_SB_.PEP0.FLD0)
                }
            }
            Method(_PS3, 0x0, NotSerialized)
            {
                Store(Buffer(ESNL)
                {
                }, DEID)
                Store(0x3, DVAL)
                Store(PGID, DEID)
                If(\_SB_.ABD_.AVBL)
                {
                    Store(DBUF, \_SB_.PEP0.FLD0)
                }
            }
        }
```
<br>

I2C controller's IC14 node:

```c
        Device (IC14)
        {
            Name (_HID, "QCOM0A10")
            Alias(\_SB.PSUB, _SUB)
            Name (_UID, 14)
            Name (_DEP, Package(){\_SB_.PEP0,\_SB_.QGP1,\_SB_.MMU0})
            Name (_CCA, 0)
            Name (_STR, Unicode ("QUP_1_SE_5"))

            Method (_CRS, 0x0, NotSerialized)
            {
                Name (RBUF, ResourceTemplate ()
                {
                    Memory32Fixed (ReadWrite, 0xa94000, 0x00004000)
                    Interrupt(ResourceConsumer, Level, ActiveHigh, Exclusive, , , ) {390} // from dts i2c node "interrupts = <0x00 0x166 0x04>;", 0x166 = 358 + 32 = 390
                })
                Return (RBUF)
            }
        }
```
<br>

PEP0 scope and it's LPXC packages:

```c
        Scope (\_SB.PEP0)
        {
            Method (LPMX, 0, NotSerialized)
            {
                Return (LPXC) /* \_SB_.PEP0.LPXC */
            }

            Name (LPXC, Package (One)
            {
                Package (0x04)
                {
                    "DEVICE",
                    "\\_SB.TSC1",
                    Package (0x06)
                    {
                        "DSTATE",
                        Zero,
                        Package (0x02)
                        {
                            "PMICVREGVOTE",
                            Package (0x08)
                            {
                                "PPP_RESOURCE_ID_LDO7_C", // Voltage Regulator ID
                                One, // Voltage Regulator type = LDO
                                0x002DC6C0, // Voltage
                                One, // Enable = Enable
                                0x07, // Power mode - Normal Power Mode
                                Zero, // Head Room
                                "HLOS_DRV",
                                "REQUIRED"
                            }
                        },

                        Package (0x02)
                        {
                            "TLMMGPIO",
                            Package (0x06)
                            {
                                0x51, // GPIO pin
                                Zero, // State low
                                Zero, // Function select 0
                                Zero, // Direction - input (input-enable)
                                Zero, // No pull (bias-disable)
                                Zero  // Drive strength - 0 == 2 mA
                            }
                        },

                        Package (0x02)
                        {
                            "TLMMGPIO",
                            Package (0x06)
                            {
                                0x51, // GPIO pin
                                Zero, // State low
                                Zero, // Function select 0
                                Zero, // Direction - input (input-enable)
                                One,  // Pull down (bias-pull-down)
                                Zero  // Drive strength - 0 == 2 mA
                            }
                        },

                        Package (0x02)
                        {
                            "DELAY",
                            Package (One)
                            {
                                0xC8
                            }
                        }
                    },

                    Package (0x05)
                    {
                        "DSTATE",
                        0x03,
                        Package (0x02)
                        {
                            "TLMMGPIO",
                            Package (0x06)
                            {
                                0x51, // GPIO pin
                                Zero, // State low
                                Zero, // Function select 0
                                Zero, // Direction - input (input-enable)
                                One,  // Pull down (bias-pull-down)
                                Zero  // Drive strength - 0 == 2 mA
                            }
                        },

                        Package (0x02)
                        {
                            "TLMMGPIO",
                            Package (0x06)
                            {
                                0x51, // GPIO pin
                                Zero, // State low
                                Zero, // Function select 0
                                Zero, // Direction - input (input-enable)
                                One,  // Pull down (bias-pull-down)
                                Zero  // Drive strength - 0 == 2 mA
                            }
                        },

                        Package (0x02)
                        {
                            "PMICVREGVOTE",
                            Package (0x07)
                            {
                                "PPP_RESOURCE_ID_LDO7_C", // Voltage Regulator ID
                                One, // Voltage Regulator type = LDO
                                Zero, // Voltage 0V
                                One, // Enable = Enable
                                0x07, // Power mode - Normal Power Mode
                                Zero, // Head Room
                                "REQUIRED"
                            }
                        }
                    }
                }
            })
        }
```
<br>

### Explanation of changes in the ACPI configuration

The `TSC1` device Hardware-ID (HID) is defined like this: `Name(_HID, "STFT556A")`, which is used by the device's Windows driver.


`IC14` is the I2C controller number that is used for touch, it is defined in the _DEP package.
```
            Name(_DEP, Package(0x3)
            {
                \_SB_.GIO0,
                \_SB_.IC14,
                \_SB_.PEP0
            })
```
