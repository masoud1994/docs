# Disater Recovery for VM on Microsoft Azure

## Backup Your Virtual Machine

Prerequisites:
* Ensure your VM is running on an Azure-supported operating system.
* Ensure you have an Azure Recovery Services Vault to store your backups.

## Steps to Backup the Virtual Machine:

1. Create a Recovery Services Vault (if you don’t already have one):

* Go to the Azure Portal.
* In the left-hand menu, click on Create a resource.
* Search for "Recovery Services Vault".
* Click on Recovery Services Vault and then click Create.
* Enter the required information:
   * Subscription: Select the subscription.
   * Resource Group: Choose an existing one or create a new one.
   * Vault Name: Give your vault a unique name.
   * Region: Choose the region closest to your VM.
* Click Review + Create and then click Create once validation passes.
2. Enable Backup for the VM:

* In the Recovery Services Vault that you created, go to the Vault and select Backup.
* In the Backup Goal, select Azure Virtual Machine.
* Under Where is your workload running?, select Azure.
* Choose Virtual Machine for the workload type and click Start.

3. Select the Virtual Machine to Back Up:

* After clicking Start, you’ll be prompted to select the VM you want to back up.
* Choose the VM from the list and click Backup.

4. Configure Backup Settings:

* Set the backup policy. Azure provides default backup policies, but you can create a custom one with your desired frequency (daily, weekly, etc.) and retention settings.
* Click Enable Backup to begin the backup process.

5. Wait for Backup Completion:

* The backup will be performed based on the configured schedule.
* You can monitor the backup process under Backup Items in the Recovery Services Vault.

* The first backup may take a longer time depending on the size of your VM.

## Restore the Virtual Machine for Testing
Once the backup is successfully completed, you can restore the VM to test the backup.

### Steps to Restore the Virtual Machine:

1. Navigate to the Recovery Services Vault:

* In the Azure Portal, go to the Recovery Services Vault where your VM backup is stored.

2. Restore a VM:

* In the Recovery Services Vault, select Backup Items under Protected Items.
* Click Azure Virtual Machine.
* You should see a list of all the VMs that have been backed up.
* Select the VM you want to restore and click on Restore VM.

3. Choose the Restore Type:*

* Restore to the original location: This will restore the VM exactly as it was, replacing the current VM (if needed).
* Restore to a new location: If you want to test the restore without affecting the current VM, you can restore it to a new VM. select this option.

4. Choose Restore Point:

* Select the Restore Point (the specific backup) that you want to restore from.
* Azure Backup will show the list of available restore points based on your backup policy (daily, weekly, etc.).

5. Configure Restore Settings:

* If you're restoring to a new VM, you can provide a new VM name and select resource group and network settings (such as the virtual network and subnet).
* You can also choose to keep the original disk names or change them for the restored VM.

* Start the Restore:

* Once all the settings are configured, click Restore to begin the process.
* It might take a while depending on the size of the VM and the restore point selected.

7. Monitor the Restore Process:

* The restoration process will be visible in the Notifications area of the Azure portal.
* You can monitor the status to know when the VM is fully restored.

8. Verify the Restored VM:

* Once the restore is complete, go to the new VM (if you restored to a new VM) and verify that the system and data are restored correctly.
* Test that applications and services on the VM are working as expected.
