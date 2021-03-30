---

copyright:
  years: 2020, 2021
lastupdated: "2021-03-04"

keywords: rotate, rotate master key, rotate encryption key, rotate root key, rotate keys automatically, key rotation, rewrap data

subcollection: hs-crypto

---

{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:external: target="_blank" .external}
{:codeblock: .codeblock}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Master key rotation
{: #master-key-rotation-intro}

After you have a master key loaded to your {{site.data.keyword.hscrypto}} instance, you can rotate the master key on demand to meet industry standards and cryptographic best practices.
{: shortdesc}

A master key is used to wrap encryption keys that are managed in the service instance. After the master key rotation, you retire the original master key and load a new master key that reencrypts the entire key storage.

## How master key rotation works
{: #how-master-key-rotation-works}

Master key rotation works by securely tranferring the value between two types of master key registers in crypto units: new master key register and current master key register. Depending on [the approach that you use to initialize your service instance](/docs/hs-crypto?topic=hs-crypto-initialize-instance-mode), the rotation process is slightly different.

The {{site.data.keyword.hscrypto}} Management Utilities and smart cards currently do not support master key rotation.
{: note}

### Rotating master keys using key part files
{: #how-master-key-rotation-works-use-files}

Before you rotate the master key using key part files, you need to create the key part files on your workstation. You can create a new master key value using either two or three key parts. To be able to rotate the master key, the current master key registers must be in `Valid` state with the same verification pattern and the new master key registers must be `Empty`.

You can rotate the master key using key part files regardless of whether your service instance has recovery crypto units assigned to it or not.

The following flow shows how master key rotation works in this mode:

1. With the `cryptounit-mk-load` command, the new master key parts are loaded to the new master key register, and the state of the new master key register is changed from `Empty` to `Full uncommitted`.
2. With the `cryptounit-mk-commit` command, the new master key is committed, and the new master key register state is changed to Full committed.
3. With the `cryptounit-mk-rotate` command:
  1. Encryption keys in key storage are decrypted by using the value in the current master key register and then reencrypted by using the value in the new master key register. The rewrapping takes place inside the hardware security module (HSM), so it's secure.
  2. The new master key is activated and loaded to the current master key register in `Valid` state, and the new master key register is cleared and back to `Empty` state.

The following chart illustrates how the master key register state changes during the master key rotation. It also shows the corresponding TKE CLI command that is used in each step. For detailed instructions, see [Rotating master keys using key part files](/docs/hs-crypto?topic=hs-crypto-rotate-master-key-cli).

![Rotating master keys by using key part files](/image/rotate_master_key.svg "How to rotate a master key by using key part files"){: caption="Figure 1. Rotating master keys using key part files" caption-side="bottom"}

### Rotating master keys using recovery crypto units
{: #how-master-key-rotation-works-recovery-crypto-unit}

If your service instance has recovery crypto units assigned to it, apart from using key part files, you can also rotate the master keys using the `ibmcloud tke auto-mk-rotate` command. With this command, a random new master key value is generated in one of the recovery crypto units for the service instance and securely moved to the other crypto units in the service instance.

Use the `ibmcloud tke auto-mk-rotate` command to rotate the master key only when your service instance has recovery crypto units assigned and PKCS #11 keystores are not enabled in your service instance. Currently, only the `us-south` and `us-east` regions are enabled with the recovery crypto units. For more information about the supported regions, see [Regions and locations](/docs/hs-crypto?topic=hs-crypto-regions).
{: important}

You don't need to prepare a new master key before the master key rotation. Before you can rotate the master key with the {{site.data.keyword.IBM_notm}} TKE CLI plug-in, all the current master key registers in both [operational crypto units](/docs/hs-crypto?topic=hs-crypto-initialize-instance-mode#understand-operational-crypto-unit) and [recovery crypto units](/docs/hs-crypto?topic=hs-crypto-initialize-instance-mode#understand-recovery-crypto-unit) should be in `Valid` state with the current master key loaded and all the new master key registers should be empty.

Comparing to the option where you load a new master key value from your local workstation files, the main difference in this mode is that a new master key value is first randomly generated within a recovery crypto unit and then exported to other crypto units.

The following flow shows how master key rotation works in this mode:

1. A new random master key value is generated in the new master key register of a recovery crypto unit assigned to the service instance. The state of the new master key register is changed from `Empty` to `Full uncommitted`.
2. The new master key value is copied to the new master key registers of the crypto units that are assigned to the service instance and other recovery crypto units for the service instance. All the new master key registers now are in `Full uncommitted` state.
3. The new master key is committed, and all the new master key registers state are changed to `Full committed`.
4. Key storage is reencrypted. Encryption keys that are managed in the service instance are decrypted using the current master key in the current master key registers and reencrypted using the new master key in the new master key registers.
5. The new master key is loaded to the current master key registers of the operational and recovery crypto units for the service instance in `Valid` state, and the new master key registers are cleared and back to `Empty` state.

For detailed instructions on how to rotate master keys by using recovery crypto units, see [Rotating master keys using recovery crypto units](/docs/hs-crypto?topic=hs-crypto-rotate-master-key-cli-recovery-crypto-unit).

## What's next
{: #master-key-rotation-next}

For more detailed instructions on options to rotate master keys, see:
- [Rotating master keys using key part files](/docs/hs-crypto?topic=hs-crypto-rotate-master-key-cli).
- [Rotating master keys using recovery crypto units](/docs/hs-crypto?topic=hs-crypto-rotate-master-key-cli-recovery-crypto-unit).