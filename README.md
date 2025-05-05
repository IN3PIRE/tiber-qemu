# Installing QEMU on Intel Tiber (TDX)

This document outlines the steps to install and configure QEMU within an Intel Trust Domain Extensions (TDX) environment. "Tiber" in this context refers to the Intel TDX platform for confidential computing.

**What is Intel Tiber/TDX?**

*   **Intel® Trust Domain Extensions (TDX):** A confidential computing technology that uses hardware-isolated virtual machines (VMs) called Trust Domains (TDs). TDX protects TDs from a broad range of software attacks, isolating them from the Virtual-Machine Manager (VMM), hypervisor, and other non-TD software on the host platform.

**Prerequisites**

*   **Supported Hardware:**  A system with 4th or 5th Gen Intel® Xeon® Scalable Processors (select SKUs with Intel® TDX) or later Intel® Xeon® processors supporting TDX. Refer to the [Canonical TDX GitHub page](https://github.com/canonical/tdx) for specific processor code names.
*   **Operating System:** Ubuntu Noble 24.04 LTS or Ubuntu Oracular 24.10 are the supported host and guest operating systems.
*   **BIOS Configuration:** Intel TDX-related settings must be enabled in your host's BIOS. These settings are crucial for the proper functioning of TDX.

**Installation Steps**

These steps are based on the [Canonical TDX GitHub repository](https://github.com/canonical/tdx).

1.  **Set up the Host OS:**

    *   Install Ubuntu Server 24.04 or 24.10.

    *   Clone the Intel TDX GitHub repository:

        ```bash
        git clone -b main https://github.com/canonical/tdx.git
        ```

    *   Run the `setup-tdx-host.sh` script:

        ```bash
        cd tdx
        sudo ./setup-tdx-host.sh
        ```

    *   Reboot.

    *   **Enable Intel TDX in the Host's BIOS:** (This is a *critical* step!)

        *   Reboot and access your BIOS settings.

        *   Enable the following settings (exact names may vary depending on your BIOS vendor):

            *   Memory Encryption (TME)
            *   Total Memory Encryption Bypass (Optional - for performance)
            *   Total Memory Encryption Multi-Tenant (TME-MT)
            *   Trust Domain Extension (TDX)
            *   TDX Secure Arbitration Mode Loader (SEAM Loader)
            *   Software Guard Extension (SGX)

    *   **Verify Intel TDX is Enabled:**

        ```bash
        dmesg | grep tdx
        ```

        *   Look for the message: `virt/tdx: module initialized`

2.  **Create a TD Image:**

    *   You can create a new TD image from scratch or convert an existing VM image.

    *   **Create a New TD Image:**

        ```bash
        cd tdx/guest-tools/image/
        sudo ./create-td-image.sh -v 24.10
        ```

        *   (Use `-v 24.04` for Ubuntu 24.04)

        *   The default root password for the created image is `123456`.

    *   **Convert a Regular VM Image:**

        *   Boot your existing Ubuntu 24.04 or 24.10 VM.

        *   Clone the Intel TDX repository *inside the VM*.

        *   Run `setup-tdx-guest.sh`:

            ```bash
            cd tdx
            sudo ./setup-tdx-guest.sh
            ```

        *   Shutdown the guest VM.

3.  **Boot the TD with QEMU:**

    *   Use the `run-td.sh` script:

        ```bash
        cd tdx/guest-tools
        ./run_td.sh
        ```

    *   Specify a custom TD image (if needed) using the `TD_IMG` environment variable:

        ```bash
        TD_IMG=/path/to/your/td-image.qcow2 ./run_td.sh
        ```

4. **Access the TD:**

    *   The `run-td.sh` script will output the SSH command to connect to the TD, usually using port `10022`.

        ```bash
        ssh -p 10022 root@localhost
        ```

5.  **Verify TDX is enabled inside the TD:**

    ```bash
    dmesg | grep tdx
    ```

    *   Look for the messages:
        *   `tdx: Guest detected`
        *   `Memory Encryption Features active: Intel TDX`

**Important Considerations and Troubleshooting**

*   **BIOS Configuration - The Foundation:** Verify all BIOS settings related to Intel TDX are configured correctly.  This is the most common source of issues. Consult your motherboard/server manual for specific details on these settings.
*   **TDX Module Version - Stay Updated:** Ensure you are using the latest TDX module version.  You can check the current version with the command `dmesg | grep "TDX module:"`. Follow the instructions on the [Canonical TDX GitHub page](https://github.com/canonical/tdx) for updating.
*   **Networking - Proxies:** If you are behind a proxy, ensure that proxy settings are configured correctly in your environment. Use `sudo -E` to preserve the user environment when running scripts with `sudo`.
*   **Performance - Right Module:** If experiencing poor performance, confirm you are using the correct TDX module version compatible with your hardware. Refer to the "Supported Hardware" table in the [Canonical TDX GitHub page](https://github.com/canonical/tdx).

**Using `virsh` (libvirt) as an Alternative**

The Intel TDX documentation also provides instructions on booting TDs using using `virsh` (libvirt). This method is recommended when you need to manage multiple TDs concurrently. See section "6.2 Boot TD with virsh (libvirt)" in the [Canonical TDX GitHub page](https://github.com/canonical/tdx)

**Remote Attestation (Optional)**

To verify the integrity of the TD's execution environment, you can set up remote attestation using Intel Tiber Trust Services. This includes setting up the Host OS and the guest OS. More information can be found at the [Canonical TDX GitHub page](https://github.com/canonical/tdx)

**Disclaimer**
This guide is based on the information available in the provided search results. Always refer to the official Intel and Ubuntu documentation for the most accurate and up-to-date information.
