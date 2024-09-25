# PIL Memory Region Settings Assignment

Based on [MTP08350 platform](https://github.com/WOA-Project/windows_silicon_qcom_lahaina/blob/26fe3908e8a651de0498eed797518ca93b2c1420/docs/PIL.md)

## SM7325 Platform ID Case Study

### Windows Firmware Information

The entire PIL region **allocated** by the UEFI firmware is:

- Start: 0x84300000
- End: 0x9BD00000
- Size: 0x17A00000

(Refer to the section named UEFI Memory Map for more information on how this is defined).

### Subsections of PIL Region from downstream device tree:

| FW Name      | CAMERA     | WPSS       | ADSP       | CDSP       | UNKNOWN0   | DHMS       | MODEM      | VENUS      | GFXUC      | UNKNOWN1   | UNKNOWN2   |
|--------------|------------|------------|------------|------------|------------|------------|------------|------------|------------|------------|------------|
| Memory Set   | Hardcoded  | PGCM       | PGCM       | PGCM       | PGCM       | PGCM       | PGCM       | PGCM       | PGCM       | PGCM       | PGCM       |
| Memory Start | 0x84300000 | 0x84800000 | 0x86100000 | 0x88900000 | 0x8A700000 | 0x8B200000 | 0x8B800000 | 0x9AE00000 | 0x9B300000 | 0x9B305000 | 0x9B309000 |
| Memory End   | 0x84800000 | 0x86100000 | 0x88900000 | 0x8A700000 | 0x8B200000 | 0x8B800000 | 0x9AE00000 | 0x9B300000 | 0x9B305000 | 0x9B309000 | 0x9BD00000 |
| Memory Size  | 0x00500000 | 0x01900000 | 0x02800000 | 0x01E00000 | 0x00B00000 | 0x00600000 | 0x0F600000 | 0x00500000 | 0x00005000 | 0x00004000 | 0x009F7000 |
| Config       | PILE       | SUB?, PILE | SUBA, PILE | SUBC, PILE |            | PILE       | SUBM       | PILE       | PILE       |    	 |            |

PGCM area is configured in PILE (qcpilEXT7280) and must match above table allocation plan.

**Below regions are hardcoded in ACPI tables / firmware and are therefore not dynamically used by the Operating System**

- CAMERA: Start 0x84300000, End 0x84800000, Size 0x00500000
	- Defined in /components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/HexagonLoader/qcpilEXT7280.inf

**Below regions are not hardcoded in ACPI tables / firmware and are therefore dynamically used by the Operating System**

For this kind of region, the PIL driver is instructed the total size of the region in use dynamically below using "PGCM":

- PGCM:	  Start 0x84800000, End 0x9BD00000, Size 0x17500000
  - Defined in /components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/HexagonLoader/qcpilEXT7280.inf

We then define every firmware binary meant to load in such region:

- WPSS:   Start 0x84800000, End 0x86100000, Size 0x01900000
  - Defined in /components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/HexagonLoader/qcpilEXT7280.inf
  - Defined in /components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/Subsystems/qcsubsys_ext_wpss7280.inf

- ADSP:   Start 0x86100000, End 0x88900000, Size 0x02800000
  - Defined in /components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/HexagonLoader/qcpilEXT7280.inf
  - Defined in /components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/Subsystems/qcsubsys_ext_adsp7280.inf

- CDSP:   Start 0x88900000, End 0x8A700000, Size 0x01E00000
	- Defined in /components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/HexagonLoader/qcpilEXT7280.inf
	- Defined in /components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/Subsystems/qcsubsys_ext_cdsp7280.inf

**UNKNOWN0 From 0x8A700000 to 0x8B200000**

- DHMS:   Start 0x8B200000, End 0x8B800000, Size 0x00600000
	- Defined in /components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/HexagonLoader/qcpilEXT7280.inf

- MODEM:  Start 0x8B800000, End 0x9AE00000, Size 0x0F600000
	- Defined in /components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/Subsystems/qcsubsys_ext_mpss7280.inf

- VENUS:  Start 0x9AE00000, End 0x9B300000, Size 0x00500000
  - Defined in /components/QC7325/Device/A52sxq/DEVICE.SOC_QC7325.A52SXQ/Extensions/Subsystems/qcsubsys_ext_mpss7280.inf

- GFXSUC: Start 0x9B300000, End 0x9B305000, Size 0x00005000
   - Defined in /components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/HexagonLoader/qcpilEXT7280.inf

**UNKNOWN1 From 0x9B305000 to 0x9B309000**

**UNKNOWN2 From 0x9B309000 to 0x9BD00000**

We reached the end of the whole reserved region in our UEFI firmware.

---

### INF Packages

/components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/HexagonLoader/qcpilEXT7280.inf

```ini
[PILReg_Common]
HKR, SubsystemLoad\DHMS, MemoryAlignment, %REG_DWORD%, 0x00100000
HKR, SubsystemLoad\VENUS, MemoryAlignment, %REG_DWORD%, 0
HKR, SubsystemLoad\GFXSUC, MemoryAlignment, %REG_DWORD%, 0x00001000

HKR, SubsystemLoad\CAMERA, MemoryReservation, %REG_DWORD%, 0x00500000
HKR, SubsystemLoad\VENUS, MemoryReservation, %REG_DWORD%, 0x00500000
HKR, SubsystemLoad\GFXSUC, MemoryReservation, %REG_DWORD%, 0x00005000
HKR, SubsystemLoad\DHMS, MemoryReservation, %REG_DWORD%, 0x00600000

HKR, SubsystemLoad\CAMERA, MemoryAddress, %REG_DWORD%, 0x84300000
HKR, SubsystemLoad\WPSS, MemoryAddress, %REG_DWORD%, 0x84800000
HKR, SubsystemLoad\ADSP, MemoryAddress, %REG_DWORD%, 0x86100000
HKR, SubsystemLoad\CDSP, MemoryAddress, %REG_DWORD%, 0x88900000
HKR, SubsystemLoad\GFXSUC, MemoryAddress, %REG_DWORD%, 0x9B300000
HKR, SubsystemLoad\MODEM, MemoryAddress, %REG_DWORD%, 0x8B800000
HKR, SubsystemLoad\VENUS, MemoryAddress, %REG_DWORD%, 0x9AE00000
;DHMS region need to be added as a region under SubsystemLoad like below as we do not want 
;PIL to use or operate on this region which is not managed by PIL by any means.
;DHMS is managed by QSM device of subsys
HKR, SubsystemLoad\DHMS, MemoryAddress, %REG_DWORD%, 0x8B200000

;0x0 - PIL-Region to be included in PGCM and usable by PIL driver.
;0x1 - PIL-Region to be excluded from PGCM and not-usable by PIL driver.
;0x2 - PIL-Region to be excluded from PGCM and to be returned to HLOS.
HKR, SubsystemLoad\MODEM, MemoryAttribute, %REG_DWORD%, 0x0
HKR, SubsystemLoad\DHMS, MemoryAttribute, %REG_DWORD%, 0x1
HKR, SubsystemLoad\ADSP, MemoryAttribute, %REG_DWORD%, 0x0
HKR, SubsystemLoad\CDSP, MemoryAttribute, %REG_DWORD%, 0x0
HKR, SubsystemLoad\GFXSUC, MemoryAttribute, %REG_DWORD%, 0x0
HKR, SubsystemLoad\VENUS, MemoryAttribute, %REG_DWORD%, 0x0
HKR, SubsystemLoad\WPSS, MemoryAttribute, %REG_DWORD%, 0x0

;Misc
HKR,PilConfig,HypProtectionEnabled,%REG_DWORD%,1
HKR,PilConfig,DoNotReturnMemoryToHLOS,%REG_DWORD%,0

;PGCM
HKR,PGCM,BaseAddress,%REG_DWORD%,0x84800000
HKR,PGCM,Size,%REG_DWORD%,0x17500000

;IMEM - this refers to PIL/reloc.Img.load.Info in ipcat - https://ipcatalog.qualcomm.com/memmap/chip/379/map/1217/version/7307/block/7971925
HKR,IMEM,BaseAddress,%REG_DWORD%,0x146AA000
HKR,IMEM,Offset,%REG_DWORD%,0x94C
```


/components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/Subsystems/qcsubsys_ext_adsp7280.inf

```ini
[PIL_Reg_common]
HKR ,SubsystemLoad\ADSP,MemoryAlignment,%REG_DWORD%,0x00100000

[PIL_Reg_MSM]
; ADSP registry values
HKR ,SubsystemLoad\ADSP,MemoryReservation,%REG_DWORD%,0x2800000
```

/components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/Subsystems/qcsubsys_ext_cdsp7280.inf

```ini
[PIL_Reg_common]
HKR ,SubsystemLoad\CDSP,MemoryAlignment,%REG_DWORD%,0x00100000

[PIL_Reg_MSM]
; CDSP registry values
HKR ,SubsystemLoad\CDSP,MemoryReservation,%REG_DWORD%,0x1e00000
```

/components/QC7325/Device/Kodiak/DEVICE.SOC_QC7325.KODIAK/Extensions/Subsystems/qcsubsys_ext_mpss7280.inf

```ini
[PIL_Reg_common]
; AMSS registry values
HKR ,SubsystemLoad\MODEM,MemoryAlignment,%REG_DWORD%,0x00400000
HKR ,SubsystemLoad\MODEM,MemoryAddress,%REG_DWORD%,0x8B800000

[PIL_Reg_MSM]
; AMSS registry values
HKR ,SubsystemLoad\MODEM,MemoryReservation,%REG_DWORD%,0xf600000

[PIL_Reg_GPS]
; GPS registry values
HKR ,SubsystemLoad\MODEM,MemoryReservation,%REG_DWORD%,0x8000000
```


### UEFI Memory Map

```c
/* ... */
{ "GPU PRR",           0x80894000, 0x00001000, AddMem, MEM_RES, WRITE_COMBINEABLE, Reserv, UNCACHED_UNBUFFERED_XN },
/* ... */
{ "MPSS_EFS",          0xA0800000, 0x00300000, AddMem, SYS_MEM, SYS_MEM_CAP, Reserv, UNCACHED_UNBUFFERED_XN },
/* ... */
{ "PIL Reserved",      0x84300000, 0x17A00000, AddMem, MEM_RES, UNCACHEABLE, Reserv, UNCACHED_UNBUFFERED_XN },
/* ... */
```
