# Ansible z/OS SMP/E zone cloning role

The Ansible role `zos_smpe_clone` will perform a sequence of steps to clone an existing SMP/E zone to a new one, considering the variables informed by the user on the specified z/OS host(s).

## Requirements

Python and Z Open Automation Utilities must be installed on the remote z/OS system, since the modules `zos_data_set`, `zos_find`, `zos_mvs_raw` and `zos_tso_command` from the collection `ibm.ibm_zos_core` are used along the role.

## Role Variables

This role has multiple variables. The descriptions and defaults for all these variables can be found in the **[`defaults/main.yml`](/defaults/main.yml)** file and **[`meta/argument_specs.yml`](/meta/argument_specs.yml)**, together with a detailed description below:

| Variable | Description | Optional? |
| -------- | ----------- | :-------: |
| **[`force_deletion`](/meta/argument_specs.yml)** | Force delete the new zone and the new data sets before trying to create them | Yes<br>(default: `false`) |
| **[`show_output`](/meta/argument_specs.yml)** | Determine if output should be displayed or not at the end | Yes<br>(default: `true`) |
| **[`smpe_datasets`](/meta/argument_specs.yml)** | List of data set names belonging to the old SMP/E zone | No |
| **[`smpe_global_csi`](/meta/argument_specs.yml)** | Data set name of the SMP/E GLOBAL CSI | No |
| **[`smpe_new_zone`](/meta/argument_specs.yml)** | Details of the new SMP/E zone | No |
| **[`smpe_old_zone`](/meta/argument_specs.yml)** | Details of the old SMP/E zone | No |
| **[`smpe_rename_dddefs`](/meta/argument_specs.yml)** | List of prefixes to be renamed on the new zone's DDDEF entries | Yes<br>(default: `[]`) |
| **[`smpe_zone_type`](/meta/argument_specs.yml)** | SMP/E zone type of the new zone | No |

### Detailed structure of variable `smpe_new_zone`:

| Variable | Attribute | Type | Optional? |
| -------- | --------- | :--: | :-------: |
| `smpe_new_zone` | `csi` | string | No |
| `smpe_new_zone` | `name` | string | No |
| `smpe_new_zone` | `prefix` | string | No |

### Detailed structure of variable `smpe_old_zone`:

| Variable | Attribute | Type | Optional? |
| -------- | --------- | :--: | :-------: |
| `smpe_old_zone` | `csi` | string | No |
| `smpe_old_zone` | `name` | string | No |
| `smpe_old_zone` | `prefix` | string | No |

### Detailed structure of each element of the optional list `smpe_rename_dddefs`:

| Variable | Attribute | Type | Optional? |
| -------- | --------- | :--: | :-------: |
| `smpe_rename_dddefs[]` | `new` | string | No |
| `smpe_rename_dddefs[]` | `old` | string | No |

## Dependencies

None.

## Example Playbook

On the scenario below, the role `zos_smpe_clone` is being used to clone the target zone `ZONEAAA` to a new zone called `ZONEBBB`. The GLOBAL zone can be found in the SMP/E CSI data set `SMPE.GLOBAL.CSI`. The DDDEF entries on the new zone `ZONEBBB` needs to get renamed from `SMPE.ZOSA` to `SMPE.ZOSB`.

The attribute `smpe_datasets` determines the list of involved data sets to be copied to a new name, considering the possible renames on `smpe_rename_dddefs`.

If any of the new renamed `smpe_datasets` and the new zone's SMP/E CSI data set exists, `force_deletion` being `true` attempts to delete the new zone from SMP/E and these data sets.

    - hosts: zos_server
      roles:
        - role: zos_smpe_clone
          smpe_global_csi: "SMPE.GLOBAL.CSI"
          smpe_datasets:
            - SMPE.ZOSA.DATASET1
            - SMPE.ZOSA.DATASET2
            - SMPE.ZOSA.DATASET3
            - SMPE.ZOSA.DATASET4
          smpe_rename_dddefs: 
            - old: SMPE.ZOSA
              new: SMPE.ZOSB
          smpe_old_zone:
            csi: SMPE.ZOSA.ZONEAAA.CSI
            name: ZONEAAA
            prefix: SMPE.ZOSA
          smpe_new_zone:
            csi: SMPE.ZOSB.ZONEBBB.CSI
            name: ZONEBBB
            prefix: SMPE.ZOSB
          smpe_zone_type: TARGET
          force_deletion: true
          show_output: true

## Sample Output

When this role is successfully executed, it will perform a sequence of steps to clone the current zone: it executes HRECALL commands to recall migrated data sets; if set to, it attempts to delete the new zone and new data sets; copy the current data sets to new ones; define and initialize the new zone's SMP/E CSI data set; copy the current zone to the new one; update the DDDEF entries considering the required renames; lists GLOBAL and the new zone.

A fact named `zos_smpe_clone_list_output` is registered, containing the output of the SMP/E LIST command executed on the GLOBAL and the new zones. It will be displayed if `show_output` is set to `true`. With it, we can confirm that both new zone and its zone index were defined correctly.

<details>
    <summary><b><i>Click to see the sample output</i></b></summary>

    "zos_smpe_clone_list_output": [
        [
            "\fPAGE 0001  - NOW SET TO GLOBAL ZONE          DATE 11/29/24  TIME 11:35:58  SMP/E 37.25   SMPLIST  OUTPUT",
            "",
            "  LIST GLOBALZONE.\fPAGE 0002  - NOW SET TO GLOBAL ZONE          DATE 11/29/24  TIME 11:35:58  SMP/E 37.25   SMPLIST  OUTPUT",
            "",
            "GLOBAL  ZONE ENTRIES",
            "",
            "",
            "  NAME",
            "",
            "GLOBAL    UPGLEVEL        = SMP/E 37.16",
            "          OPTIONS         = BUILD",
            "          ZONEINDEX       = DLIB     DLIB     SMPE.ZOS.DLIB.CSI",
            "                            ZONEAAA  TARGET   SMPE.ZOSA.ZONEAAA.CSI",
            "                            ZONEBBB  TARGET   SMPE.ZOSB.ZONEBBB.CSI",
            "          SREL            = Z038\fPAGE 0003  - NOW SET TO GLOBAL ZONE          DATE 11/29/24  TIME 11:35:58  SMP/E 37.25   SMPLIST  OUTPUT",
            "",
            "LIST     SUMMARY REPORT FOR GLOBAL",
            "",
            "ENTRY-TYPE   ENTRY-NAME         STATUS",
            "",
            "ZONE                            STARTS ON PAGE 0002",
            ""
        ],
        [
            "\fPAGE 0001  - NOW SET TO TARGET ZONE ZONEBBB  DATE 11/29/24  TIME 11:36:13  SMP/E 37.25   SMPLIST  OUTPUT",
            "",
            "  LIST TARGETZONE",
            "      DDDEF.\fPAGE 0002  - NOW SET TO TARGET ZONE ZONEBBB  DATE 11/29/24  TIME 11:36:13  SMP/E 37.25   SMPLIST  OUTPUT",
            "",
            "ZONEBBB ZONE ENTRIES",
            "",
            "",
            "  NAME",
            "",
            "ZONEBBB   TZONE           = ZONEBBB",
            "          UPGLEVEL        = SMP/E 37.16",
            "          RELATED         = DLIB",
            "          OPTIONS         = BUILD",
            "          SREL            = Z038\fPAGE 0003  - NOW SET TO TARGET ZONE ZONEBBB  DATE 11/29/24  TIME 11:36:13  SMP/E 37.25   SMPLIST  OUTPUT",
            "",
            "ZONEBBB DDDEF ENTRIES",
            "",
            "",
            "  NAME",
            "",
            "AADSN1    DATASET         = SMPE.ZOS.ADSN1",
            "          UNIT            = SYSALLDA",
            "          SHR",
            "          WAIT",
            "",
            "AADSN2    DATASET         = SMPE.ZOS.ADSN2",
            "          UNIT            = SYSALLDA",
            "          SHR",
            "          WAIT",
            "",
            "SADMMOD   DATASET         = SYS1.SADMMOD",
            "          UNIT            = SYSALLDA",
            "          SHR",
            "          WAIT",
            "",
            "SCEELKED  DATASET         = SYS1.SCEELKED",
            "          UNIT            = SYSALLDA",
            "          SHR",
            "          WAIT",
            "",
            "SCEESPC   DATASET         = SYS1.SCEESPC",
            "          UNIT            = SYSALLDA",
            "          SHR",
            "          WAIT",
            "",
            "DATASET1  DATASET         = SMPE.ZOSB.DATASET1",
            "          UNIT            = SYSALLDA",
            "          SHR",
            "          WAIT",
            "",
            "DATASET2  DATASET         = SMPE.ZOSB.DATASET2",
            "          UNIT            = SYSALLDA",
            "          SHR",
            "          WAIT",
            "",
            "DATASET3  DATASET         = SMPE.ZOSB.DATASET3",
            "          UNIT            = SYSALLDA",
            "          SHR",
            "          WAIT",
            "",
            "DATASET4  DATASET         = SMPE.ZOSB.DATASET4",
            "          UNIT            = SYSALLDA",
            "          SHR",
            "          WAIT",
            "",
            "SMPLOG    DATASET         = SMPE.ZOSB.ZONEBBB.SMPLOG",
            "          UNIT            = SYSALLDA",
            "          MOD",
            "          WAIT",
            "",
            "SMPLOGA   DATASET         = SMPE.ZOSB.ZONEBBB.SMPLOGA",
            "          UNIT            = SYSALLDA",
            "          MOD",
            "          WAIT",
            "",
            "SMPLTS    DATASET         = SMPE.ZOSB.ZONEBBB.SMPLTS",
            "          UNIT            = SYSALLDA",
            "          OLD",
            "          WAIT",
            "",
            "SMPMTS    DATASET         = SMPE.ZOSB.ZONEBBB.SMPMTS",
            "          UNIT            = SYSALLDA",
            "          OLD",
            "          WAIT",
            "",
            "SMPPTS    DATASET         = SMPE.ZOS.SMPPTS",
            "          UNIT            = SYSALLDA",
            "          SHR",
            "          WAIT",
            "",
            "SMPSCDS   DATASET         = SMPE.ZOSB.ZONEBBB.SMPSCDS",
            "          UNIT            = SYSALLDA",
            "          OLD",
            "          WAIT",
            "",
            "SMPSTS    DATASET         = SMPE.ZOSB.ZONEBBB.SMPSTS",
            "          UNIT            = SYSALLDA",
            "          OLD",
            "          WAIT",
            "",
            "LIST     SUMMARY REPORT FOR ZONEBBB",
            "",
            "ENTRY-TYPE   ENTRY-NAME         STATUS",
            "",
            "DDDEF                           STARTS ON PAGE 0003",
            "ZONE                            STARTS ON PAGE 0002",
            ""
        ]
    ]

</details>

## License

This role is licensed under licensed under [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0).

## Author Information

This role was created in 2024 by Luiggi Torricelli.
