# netics-oprec-module2-2026

> Nama: Jalu Cahyo Senodiputro
> NRP: 5025241155

#### Instalasi Wazuh Manager pada VPS Ubuntu 24.04 Microsoft Azure
Untuk setup Wazuh Manager di VPS sebenarnya cukup gampang, langkah-langkahnya kurang lebih seperti ini:
1. Buat VM Ubuntu 24.04 dari Microsoft Azure dengan Minimal 2 CPU Cores & 4 GB RAM
2. Lalu kita harus membuat *rule port* yang akan digunakan Wazuh Manager (1514, 1515, (*opsional* 55000)) dengan cara pergi ke `Network Settings -> Create port rule` dari Dashboard VM
3. Lakukan Instalasi Wazuh Manager dengan mengikuti tutorial dari [Wazuh Manager Installation](https://documentation.wazuh.com/current/quickstart.html)
4. Terakhir kita bisa Login ke Web Dashboard Wazuh berdasarkan petunjuk dari instalasi diatas
    ```
    INFO: --- Summary ---
    INFO: You can access the web interface https://<WAZUH_DASHBOARD_IP_ADDRESS>
        User: admin
        Password: <ADMIN_PASSWORD>
    INFO: Installation finished.
    ```

#### Instalasi Wazuh Agent pada VM Local & VPS Microsoft Azure Ubuntu 24.04
Disini saya memilih untuk melakukan instalasi Wazuh Agent di 2 mesin berbeda, 1 di VPS yang sebelumnya dibuat untuk pengerjaan modul 1 dan 1 nya saya buat di VM Local.

Untuk proses instalasinya cukup gampang juga, langkah-langkahnya yaitu seperti ini:
1. Persiapkan mesin atau host yang akan kita instalasi Wazuh Agentnya
2. Ikuti tutorial instalasi dari [Wazuh Agent Installation](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html). Pada step **Deploy a Wazuh agent** sesuaikan `WAZUH_MANAGER` dengan IP Address Wazuh Manager atau bisa juga mengikuti tutorial ini [Wazuh Agent Configuration](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/enrollment-methods/via-agent-configuration/linux-endpoint.html)

#### Pastikan agent berhasil terhubung ke manager dengan mengecek apakah default events/logs dari agent sudah masuk dan terlihat di dashboard Wazuh.
Jika Wazuh Agent berhasil terinstal dan berjalan maka kita bisa kembali ke web dashboard Wazuh Manager untuk mengecek apakah Agent tersebut sudah berhasil terhubung ke Manager
![alt text](assets/agent-list.png)
![alt text](assets/go-health-app-summary.png)
![alt text](assets/gryd-wazuh-summary.png)


#### Buat/mencari dan implementasikan minimal 5 custom rules/custom alert pada agent kalian. Jenis rules bebas dan dapat menyesuaikan kreativitas serta konteks keamanan masing-masing. Maksimal rules tidak dibatasi, lebih banyak lebih baik.
```
<group name="local,syslog,sshd,custom,journald">

        <!--
                Dec 10 01:02:02 host sshd[1234]: Failed none for root from 1.1.1.1 port 1066 ssh2
        -->
        <rule id="100001" level="5">
                <if_sid>5716</if_sid>
                <srcip>1.1.1.1</srcip>
                <description>sshd: authentication failed from IP 1.1.1.1.</description>
                <group>authentication_failed,pci_dss_10.2.4,pci_dss_10.2.5,</group>
        </rule>

        <rule id="110002" level="5">
                <if_sid>5715</if_sid>
                <if_group>syslog</if_group>
                <match>azureuser</match>
                <description>Successful SSH login as azureuser.</description>
                <mitre><id>T1021</id></mitre>
        </rule>

        <rule id="110003" level="10">
                <if_group>syslog</if_group>
                <match>sudo:</match>
                <description>Privilege escalation activity: sudo command executed (check user and command).</description>
                <mitre><id>T1548</id></mitre>
        </rule>

        <rule id="110004" level="9">
                <if_group>syslog</if_group>
                <match>session opened for user root</match>
                <description>Privilege escalation activity: session opened for user root (possible su/sudo).</description>
                <mitre><id>T1548</id></mitre>
        </rule>

        <rule id="110006" level="12">
                <if_sid>2504</if_sid>
                <if_group>syslog</if_group>
                <match>ROOT LOGIN REFUSED</match>
                <description>SSH failed attempt using root account.</description>
                <mitre><id>T1110</id></mitre>
        </rule>

        <rule id="110007" level="12">
                <if_group>syslog</if_group>
                <regex>sshd\p\d+\p\p\sAccepted\p \w+\p for root\p from \d+\.\d+\.\d+\.\d+</regex>
                <description>SSH accepted login as root (should be disabled).</description>
                <mitre><id>T1110</id></mitre>
        </rule>
</group>
```