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

| FW Name      | CAMERA     | WPSS       | ADSP       | CDSP       | UNKNOWN    | DHMS       | MODEM      | VENUS      | GFXUC      | UNKNOWN    | UNKNOWN    |
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

- PGCM:   Start 0x84800000, End 0x9BD00000, Size 0x17500000
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

(unfinished from here, also not all addresss above are corrected)

**UNKNOWN1 From 0x8A700000 to 0x8B200000**

- DHMS:   Start 0x8A600000, End 0x8A610000, Size 0x00010000
	- Defined in /components/QC7325/Device/A52sxq/DEVICE.SOC_QC7325.A52SXQ/Extensions/HexagonLoader/qcpilEXT7280.inf

- MODEM:  Start 0x8A61A000, End 0x8A61C000, Size 0x00002000
	- Defined in /components/QC7325/Device/A52sxq/DEVICE.SOC_QC7325.A52SXQ/Extensions/HexagonLoader/qcpilEXT7280.inf

- VENUS:  Start 0x8B700000, End 0x9B700000, Size 0x10000000
	- Defined in /components/QC7325/Device/A52sxq/DEVICE.SOC_QC7325.A52SXQ/Extensions/Subsystems/qcsubsys_ext_mpss7280.inf

- GFXSUC:   Start 0x9B700000, End 0x9BD00000, Size 0x00600000
  - Defined in /components/QC7325/Device/A52sxq/DEVICE.SOC_QC7325.A52SXQ/Extensions/HexagonLoader/qcpilEXT7280.inf

**UNKNOWN2 From 0x9B305000 to 0x9B309000**

**UNKNOWN3 From 0x9B309000 to 0x9BD00000**

We reached the end of the whole reserved region in our UEFI firmware.

---

### INF Packages

/components/QC7325/Device/A52sxq/DEVICE.SOC_QC7325.A52SXQ/Extensions/HexagonLoader/qcpilEXT7280.inf

```ini
[PILReg_Common]
HKR ,SubsystemLoad\VENUS,MemoryAlignment,%REG_DWORD%,0
HKR ,SubsystemLoad\GFXSUC,MemoryAlignment,%REG_DWORD%,0x00001000
; Venus registry values
HKR ,SubsystemLoad\VENUS,MemoryReservation,%REG_DWORD%,0x00500000
; GFX registry values
; Original size is 0x00005000, not sure about this
HKR ,SubsystemLoad\GFXSUC,MemoryReservation,%REG_DWORD%,0x00002000

; ---
; CAMERA registry values
HKR, SubsystemLoad\CAMERA, MemoryAddress,     %REG_DWORD%, 0x84300000
HKR, SubsystemLoad\CAMERA, MemoryReservation, %REG_DWORD%, 0x00500000
; Venus registry values
HKR, SubsystemLoad\VENUS, MemoryAddress, %REG_DWORD%, 0x84800000
; EVA registry values
HKR, SubsystemLoad\EVA, MemoryAddress,     %REG_DWORD%, 0x84D00000
HKR, SubsystemLoad\EVA, MemoryReservation, %REG_DWORD%, 0x00500000
; ADSP registry values
HKR, SubsystemLoad\ADSP, MemoryAddress, %REG_DWORD%, 0x85200000
; CDSP registry values
HKR, SubsystemLoad\CDSP, MemoryAddress, %REG_DWORD%, 0x88800000
; IPA registry values
HKR, SubsystemLoad\IPA, MemoryAddress,     %REG_DWORD%, 0x8A600000
HKR, SubsystemLoad\IPA, MemoryReservation, %REG_DWORD%, 0x00010000
; GFX registry values
HKR, SubsystemLoad\GFXSUC, MemoryAddress, %REG_DWORD%, 0x8A61A000
; ---

;DHMS region need to be added as a region under SubsystemLoad like below as we do not want 
;PIL to use or operate on this region which is not managed by PIL by any means.
;DHMS is managed by QSM device of subsys
HKR,SubsystemLoad\DHMS,MemoryAddress,%REG_DWORD%,0x9B700000	
HKR,SubsystemLoad\DHMS,MemoryReservation,%REG_DWORD%,0x00600000
HKR,SubsystemLoad\DHMS,MemoryAlignment,%REG_DWORD%,0x00600000
;0x0 - PIL-Region to be included in PGCM and usable by PIL driver.
;0x1 - PIL-Region to be excluded from PGCM and not-usable by PIL driver.
;0x2 - PIL-Region to be excluded from PGCM and to be returned to HLOS.
HKR,SubsystemLoad\DHMS,MemoryAttribute,%REG_DWORD%,0x1

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


/components/QC7325/Device/A52sxq/DEVICE.SOC_QC7325.A52SXQ/Extensions/Subsystems/qcsubsys_ext_adsp7280.inf

```ini
[PIL_Reg_common]
HKR ,SubsystemLoad\ADSP,MemoryAlignment,%REG_DWORD%,0x00100000

[PIL_Reg_MSM]
; ADSP registry values
HKR ,SubsystemLoad\ADSP,MemoryReservation,%REG_DWORD%,0x2100000
```

/components/QC7325/Device/A52sxq/DEVICE.SOC_QC7325.A52SXQ/Extensions/Subsystems/qcsubsys_ext_cdsp7280.inf

```ini
[PIL_Reg_common]
HKR ,SubsystemLoad\CDSP,MemoryAlignment,%REG_DWORD%,0x00100000

[PIL_Reg_MSM]
; CDSP registry values
HKR ,SubsystemLoad\CDSP,MemoryReservation,%REG_DWORD%,0x1e00000
```

/components/QC7325/Device/A52sxq/DEVICE.SOC_QC7325.A52SXQ/Extensions/Subsystems/qcsubsys_ext_mpss7280.inf

```ini
[PIL_Reg_common]
; AMSS registry values
HKR ,SubsystemLoad\MODEM,MemoryAlignment,%REG_DWORD%,0x00400000
HKR ,SubsystemLoad\MODEM,MemoryAddress,%REG_DWORD%,0x8B700000

[PIL_Reg_MSM]
; AMSS registry values
HKR ,SubsystemLoad\MODEM,MemoryReservation,%REG_DWORD%,0x10000000

[PIL_Reg_GPS]
; GPS registry values
HKR ,SubsystemLoad\MODEM,MemoryReservation,%REG_DWORD%,0x8000000
```

(not edited from now on)

### UEFI Memory Map

```c
/* ... */
{"GPU PRR",           0x80880000, 0x00010000, AddMem, MEM_RES, WRITE_COMBINEABLE, Reserv, UNCACHED_UNBUFFERED_XN},
/* ... */
{"MPSS_EFS",          0x82480000, 0x00280000, AddMem, SYS_MEM, SYS_MEM_CAP, Reserv, UNCACHED_UNBUFFERED_XN},
/* ... */
{"PIL Reserved",      0x85200000, 0x16C00000, AddMem, MEM_RES, UNCACHEABLE, Reserv, UNCACHED_UNBUFFERED_XN},
/* ... */
```

### Android Firmware Information

#### Device Tree

```dts
hyp_mem: hyp_region@80000000 {
	no-map;
	reg = <0x0 0x80000000 0x0 0x600000>;
};
 
xbl_aop_mem: xbl_aop_region@80700000 {
	no-map;
	reg = <0x0 0x80700000 0x0 0x160000>;
};

xbl_apps_hob_mem: xbl_apps_hob_mem@e3400000 {
	no-map;
	reg = <0x0 0xe3400000 0x0 0x1000>;
};

pon_info_hob_mem: pon_info_hob_mem@e3401000 {
	no-map;
	reg = <0x0 0xe3401000 0x0 0x1000>;
};

cmd_db: reserved-memory@80860000 {
	compatible = "qcom,cmd-db";
	no-map;
	reg = <0x0 0x80860000 0x0 0x20000>;
};

reserved_xbl_uefi_log: res_xbl_uefi_log_region@80880000 {
	no-map;
	reg = <0x0 0x80880000 0x0 0x14000>;
};

smem_mem: smem_region@80900000 {
	no-map;
	reg = <0x0 0x80900000 0x0 0x200000>;
};

cpucp_fw_mem: cpucp_fw_region@80b00000 {
	no-map;
	reg = <0x0 0x80b00000 0x0 0x100000>;
};

cdsp_secure_heap: cdsp_secure_heap@80c00000 {
	no-map;
	reg = <0x0 0x80c00000 0x0 0x4600000>;
};

pil_camera_mem: pil_camera_region@85200000 {
	no-map;
	reg = <0x0 0x85200000 0x0 0x500000>;
};

pil_video_mem: pil_video_region@85700000 {
	no-map;
	reg = <0x0 0x85700000 0x0 0x500000>;
};

pil_cvp_mem: pil_cvp_region@85c00000 {
	no-map;
	reg = <0x0 0x85c00000 0x0 0x500000>;
};

pil_adsp_mem: pil_adsp_region@86100000 {
	no-map;
	reg = <0x0 0x86100000 0x0 0x2100000>;
};

pil_slpi_mem: pil_slpi_region@88200000 {
	no-map;
	reg = <0x0 0x88200000 0x0 0x1500000>;
};

pil_cdsp_mem: pil_cdsp_region@89700000 {
	no-map;
	reg = <0x0 0x89700000 0x0 0x1e00000>;
};

pil_ipa_fw_mem: pil_ipa_fw_region@8b500000 {
	no-map;
	reg = <0x0 0x8b500000 0x0 0x10000>;
};

pil_ipa_gsi_mem: pil_ipa_gsi_region@8b510000 {
	no-map;
	reg = <0x0 0x8b510000 0x0 0xa000>;
};

pil_gpu_mem: pil_gpu_region@8b51a000 {
	no-map;
	reg = <0x0 0x8b51a000 0x0 0x2000>;
};

pil_spss_mem: pil_spss_region@8b600000 {
	no-map;
	reg = <0x0 0x8b600000 0x0 0x100000>;
};

pil_modem_mem: modem_region@8b800000 {
	no-map;
	reg = <0x0 0x8b800000 0x0 0x10000000>;
};

hyp_reserved_mem: qheebsp_dbg_vm_hyp_region@d0000000 {
	no-map;
	reg = <0x0 0xd0000000 0x0 0x800000>;
};

pil_trustedvm_mem: pil_trustedvm_region@d0800000 {
	no-map;
	reg = <0x0 0xd0800000 0x0 0x76f7000>;
};

qrtr_shbuf: qrtr-shmem {
	no-map;
	reg = <0x0 0xd7ef7000 0x0 0x9000>;
};

chan0_shbuf: neuron_block@0 {
	no-map;
	reg = <0x0 0xd7f00000 0x0 0x80000>;
};

chan1_shbuf: neuron_block@1 {
	no-map;
	reg = <0x0 0xd7f80000 0x0 0x80000>;
};

removed_mem: removed_region@d8800000 {
	no-map;
	reg = <0x0 0xd8800000 0x0 0x6800000>;
};

adsp_mem: adsp_region {
	compatible = "shared-dma-pool";
	alloc-ranges = <0x0 0x00000000 0x0 0xffffffff>;
	reusable;
	alignment = <0x0 0x400000>;
	size = <0x0 0xC00000>;
};

sdsp_mem: sdsp_region {
	compatible = "shared-dma-pool";
	alloc-ranges = <0x0 0x00000000 0x0 0xffffffff>;
	reusable;
	alignment = <0x0 0x400000>;
	size = <0x0 0x800000>;
};

cdsp_mem: cdsp_region {
	compatible = "shared-dma-pool";
	alloc-ranges = <0x0 0x00000000 0x0 0xffffffff>;
	reusable;
	alignment = <0x0 0x400000>;
	size = <0x0 0x400000>;
};

user_contig_mem: user_contig_region {
	compatible = "shared-dma-pool";
	alloc-ranges = <0x0 0x00000000 0x0 0xffffffff>;
	reusable;
	alignment = <0x0 0x400000>;
	size = <0x0 0x1000000>;
};

qseecom_mem: qseecom_region {
	compatible = "shared-dma-pool";
	alloc-ranges = <0x0 0x00000000 0x0 0xffffffff>;
	reusable;
	alignment = <0x0 0x400000>;
	size = <0x0 0x1400000>;
};

qseecom_ta_mem: qseecom_ta_region {
	compatible = "shared-dma-pool";
	alloc-ranges = <0x0 0x00000000 0x0 0xffffffff>;
	reusable;
	alignment = <0x0 0x400000>;
	size = <0x0 0x1000000>;
};

secure_display_memory: secure_display_region { 
	compatible = "shared-dma-pool";
	alloc-ranges = <0x0 0x00000000 0x0 0xffffffff>;
	reusable;
	alignment = <0x0 0x400000>;
	size = <0x0 0xA400000>;
};

cnss_wlan_mem: cnss_wlan_region {
	compatible = "shared-dma-pool";
	alloc-ranges = <0x0 0x00000000 0x0 0xffffffff>;
	reusable;
	alignment = <0x0 0x400000>;
	size = <0x0 0x1400000>;
};


qcom: ramoops {
	compatible = "ramoops";
	reg = <0x0 0xa9000000 0x0 0x200000>;
	pmsg-size = <0x200000>;
	mem-type = <2>;
};


linux,cma {
	compatible = "shared-dma-pool";
	alloc-ranges = <0x0 0x00000000 0x0 0xdfffffff>;
	reusable;
	alignment = <0x0 0x400000>;
	size = <0x0 0x2000000>;
	linux,cma-default;
};

dump_mem: mem_dump_region {
	compatible = "shared-dma-pool";
	alloc-ranges = <0x0 0x00000000 0x0 0xffffffff>;
	reusable;
	size = <0 0x3000000>;
};

sp_mem: sp_region {  
	compatible = "shared-dma-pool";
	alloc-ranges = <0x0 0x00000000 0x0 0xffffffff>;
	reusable;
	alignment = <0x0 0x400000>;
	size = <0x0 0x1000000>;
};

audio_cma_mem: audio_cma_region {
	compatible = "shared-dma-pool";
	alloc-ranges = <0x0 0x00000000 0x0 0xffffffff>;
	reusable;
	alignment = <0x0 0x400000>;
	size = <0x0 0x1C00000>;
};

splash_memory:splash_region {
	reg = <0x0 0xe1000000 0x0 0x02300000>;
	label = "cont_splash_region";
};


demura_memory_0: demura_region_0 {
	reg = <0x0 0x0 0x0 0x0>;
	label = "demura hfc region 0";
};

demura_memory_1: demura_region_1 {
	reg = <0x0 0x0 0x0 0x0>;
	label = "demura hfc region 1";
};

dfps_data_memory: dfps_data_region {
	reg = <0x0 0xe3300000 0x0 0x0100000>;
	label = "dfps_data_region";
};

memshare_mem: memshare_region {
	compatible = "shared-dma-pool";
	no-map;
	
	alloc-ranges = <0x0 0x00000000 0x0 0xdfffffff>;
	alignment = <0x0 0x100000>;
	size = <0x0 0x800000>;
};

non_secure_display_memory: non_secure_display_region {
	compatible = "shared-dma-pool";
	alloc-ranges = <0x0 0x00000000 0x0 0xffffffff>;
	reusable;
	alignment = <0x0 0x400000>;
	size = <0x0 0x6400000>;
};
```

#### XBL

```ini
0x85200000, 0x16600000, "PIL Reserved",      AddMem, MEM_RES, UNCACHEABLE, Reserv, UNCACHED_UNBUFFERED_XN
```

---

_**© 2020-2024 The Duo WOA Authors**_

_Snapdragon is a registered trademark of Qualcomm Incorporated. Microsoft, the Microsoft Corporate Logo, Windows, Surface, Surface Duo, Windows Hello, Continuum, Hyper-V, and DirectX are registered trademarks of Microsoft Corporation in the United States. Android is a registered trademark of Google LLC. Miracast is a registered trademark of the Wi-Fi Alliance. Other binaries may be copyright Qualcomm Incorporated and Microsoft Surface._

_**Limited emergency calling**_

_Running Windows on your Surface Duo is not a replacement for a proper phone operating system and does not have emergency calling capabilities._

_**Hello from Seattle (US), France, Italy.**_
