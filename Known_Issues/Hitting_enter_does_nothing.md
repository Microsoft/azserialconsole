## Issue ##
When trying to use Azure Serial Console to connect to a Linux virtual machine you see the banner below but hitting enter does not do anything.  The login prompt is not displayed and the console is not responsive.

![](../images/PressEnter.jpg)

## Cause ##
This issue occurs when the serial console connection is successfully made to the virtual machine, but the virtual machine is not configured for input/output on the serial port.

## More Info ##
Virtual machines published by [endorsed distribution](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/endorsed-distros) partners to provision in Azure will already be configured to enable serial console.  However, if a virtual machine was uploaded to Azure or created from a custom image the necessary settings may not be in place.

_Note: certain VM misconfigurations may also cause failures to capture [boot diagnostics](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/boot-diagnostics) logging via serial output._


## Resolution ##
#### Linux Guest ####
To ensure your Linux virtual machine works with Azure serial console, please follow the guidance: [Create and upload a Linux VHD in Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-generic) for the distribution you are using. Specifically you will need to perform these steps from the preceding link:

>Modify the kernel boot line in GRUB or GRUB2 to include the following parameters. This will also ensure all console messages are sent to the first serial port, which can assist Azure support with debugging issues:
>
>        console=ttyS0,115200 earlyprintk=ttyS0,115200 rootdelay=300
>  
>    In addition to the above, it is recommended to *remove* the following parameters if they exist:
>  
>        rhgb quiet crashkernel=auto
>  
>    Graphical and quiet boot are not useful in a cloud environment where we want all the logs to be sent to the serial port. The `crashkernel` option may be left configured if desired, but note that this parameter will reduce the amount of available memory in the VM by 128MB or more, which may be problematic on the smaller VM sizes.

#### Windows Guest ####
Ensure you have enabled the [Special Administrative Console](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/serial-console#accessing-serial-console-for-windows).

