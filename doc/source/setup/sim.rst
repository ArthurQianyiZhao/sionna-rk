.. _sim:

SIM Card Programming
====================

To connect commercial-off-the-shelf (COTS) user equipment (UE), a SIM card needs to be programmed with specific parameters. This guide walks through programming SIM cards using the `Open Cells <https://open-cells.com>`_ UICC programming tool, which allows configuring the necessary authentication and identification parameters for network access.

Prerequisites
-------------

You will need the `UICC software and a USB SIM card programmer <https://open-cells.com/index.php/uiccsim-programing/>`_ from Open Cells. Ensure ``make`` and ``gcc`` are installed.

Since Open Cells has stopped providing SIM cards and the corresponding readers, an alternative is to use SIM cards from `Sysmocom <https://shop.sysmocom.de/sysmoISIM-SJA5-9FV-SIM-USIM-ISIM-Card-10-pack-with-ADM-keys-9FV-chip/sysmoISIM-SJA5-9FV-10p-adm>`_ and the `pysim <https://github.com/osmocom/pysim>`_ programming tool.

UICC Software Setup
-------------------

The following steps can be done on the Ubuntu host machine.

Download `UICC v3.3 <https://open-cells.com/d5138782a8739209ec5760865b1e53b0/uicc-v3.3.tgz>`_ programming tool:

.. code-block:: bash

   wget https://open-cells.com/d5138782a8739209ec5760865b1e53b0/uicc-v3.3.tgz

Extract and compile:

.. code-block:: bash

   # Extract package
   tar xzf uicc-v3.3.tgz

   # Compile
   make clean
   make program_uicc
   make program_uicc_pcsc

Programming SIM Card
--------------------

Insert the SIM card and connect the USB programmer.
Read current values:

.. code-block:: bash

   sudo ./program_uicc --adm 1

Example output:

.. code-block:: text

   Existing values in USIM
   ICCID: 89330061100000000831
   WARNING: iccid luhn encoding of last digit not done
   USIM IMSI: 208920100001831
   PLMN selector: : 0x02f8297c
   Operator Control PLMN selector: : 0x02f8297c
   Home PLMN selector: : 0x02f8297c
   USIM MSISDN: 00000831
   USIM Service Provider Name: open cells
   No ADM code of 8 figures, can't program the UICC

Now let's program the SIM card with the network parameters.

.. note::
   The parameters (``key``, ``opc``, ``spn``) must match your core network configuration, particularly the Access Management Function (AMF) settings. These values should align with the core network setup defined in the AMF. In OAI tutorials, these parameters are typically configured in an SQL file that initializes the network. Make sure the ``key``, ``opc``, and ``spn`` values match your network configuration, and verify that the IMSI number is registered in your OAI database (``config/common/oai_db.sql``).

In this example, 262 represents Germany, and 99 is unassigned. These parameters are used for a test UE in the OAI tutorial setup.

.. code-block:: bash

   sudo ./program_uicc --adm 12345678 \
       --imsi 262990100016069 \
       --isdn 00000001 \
       --acc 0001 \
       --key fec86ba6eb707ed08905757b1bb44b8f \
       --opc C42449363BBAD02B66D16BC975D77CC1 \
       --spn "OpenAirInterface" \
       --authenticate


Example output:

.. code-block:: text

   Setting new values

   Reading UICC values after uploading new values
   ICCID: 89330061100000000831
   USIM IMSI: 262990100016069
   PLMN selector: : 0x02f8997c
   USIM MSISDN: 00000001
   USIM Service Provider Name: OpenAirInterface
   Succeeded to authenticate with SQN: 352
   Set HSS SQN value as: 384


Alternatively, program a sysmoISIM card with the `pySim-prog <https://osmocom.org/projects/pysim/wiki/PySim-prog>`_ tool. The ADM key is provided with the sysmoISIM card. We use a standard PC/SC reader here as an example. Serials readers could also be used with different command line options.

Install `pysim <https://github.com/osmocom/pysim>`_ for programming a sysmoISIM card:

.. code-block:: bash

   git clone https://gitea.osmocom.org/sim-card/pysim.git
   cd pysim
   python3 -m pip install -r requirements.txt

Read the card with PC/SC reader:

.. code-block:: bash

   ./pySim-read.py -p 0

Program the sysmoISIM card with the same network parameters. Note that the ADM value must match the provided value from vendor, otherwise it may lock the card for writing privileges.

.. code-block:: bash

   ./pySim-prog.py -p 0 \
       -a 12345678 \
       -i 262990100016069 \
       --acc=0001 \
       -s 89330061100000000831 \
       -k fec86ba6eb707ed08905757b1bb44b8f \
       -o C42449363BBAD02B66D16BC975D77CC1 \
       -n OpenAirInterface \
       --msisdn 00000001

Example output:

.. code-block:: text

   INFO: Using PC/SC reader interface
   Ready for Programming: Insert card now (or CTRL-C to cancel)
   Autodetected card type: sysmoUSIM-SJA5
   Generated card parameters :
   > Name     : OpenAirInterface
   > SMSP     : e1ffffffffffffffffffffffff0581005155f5ffffffffffff000000
   > ICCID    : 89330061100000000831
   > MCC/MNC  : 901/55
   > IMSI     : 262990100016069
   > Ki       : fec86ba6eb707ed08905757b1bb44b8f
   > OPC      : C42449363BBAD02B66D16BC975D77CC1
   > ACC      : 0001
   > ADM1(hex): 3132333435363738
   > OPMODE   : None
   Programming successful: Remove card from reader



Additional Resources
--------------------

* `UICC Programming Guide <https://open-cells.com/index.php/uiccsim-programing/>`_
* `PLMN Information <https://en.wikipedia.org/wiki/Public_land_mobile_network>`_
* `Mobile Network Codes <https://en.wikipedia.org/wiki/Mobile_country_code>`_
* `pySim-programming tool <https://github.com/osmocom/pysim>`_
* `PySim-prog guide <https://osmocom.org/projects/pysim/wiki/PySim-prog>`_
