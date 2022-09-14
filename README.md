# How to create a free Oracle VPS with Python script (Out of capacity)?

## Introduction
Oracle allow us to create free resources (such as VM) at their cloud services for testing. For examples, a VM.Standard.A1.Flex with maximal 4 OCPU and 24GB RAM is a good deal to host a website. However, the deal isn’t always available because the demand is always more then what Oracle can provide. If you try to create a free VM for now, you’ll receive an error “Out of capacity” most of the time. In this post, I’ll show you how to run a script which runs over the time and tries to create a VM until it gets success.

## Steps
- The script is a Python script. You need to install Python on your computer. The script needs some settings so that it works properly. The values of variables can be get from following steps.
- Go to Oracle Cloud and open an account. You’ll need a master card for it. The `Home region` is where you want to create your VM. The `Home region` can’t be changed after set. So choose your region wisely so that you can take advantage of the VM later.
- After creating an account, login to your account. On the the top right, click the profile icon and then select `User Settings`.

![image](https://user-images.githubusercontent.com/58999917/190090909-785f8de2-ce2d-4e57-ab7f-8e8eeb6c8b0a.png)

- Scroll down to `Resources`. Select `API Keys`. Click `Add API Key`.

![image](https://user-images.githubusercontent.com/58999917/190091050-d3425fc0-186c-42a3-9e16-49744b4d0442.png)

- Make sure `Generate API Key Pair` is selected, click `Download Private Key` and save it as file `oci_private_key.pem`.
- Click `Add`. Copy the contents from the textarea and save it as file `config`.
- Put the `config` and `oci_private_key.pem` files together in same directory.
- Open the config file and edit the row of `key_file`. That is the path to your private key. NOTE: If you use Windows, the path must be an absolute path.
```
key_file=oci_private_key.pem
```
- Go to Oci Python script repository and download file `oci_auto.py`. Save it in the directory with the other files.
- Go back to Oracle Cloud and `Create a VM instance`.

![image](https://user-images.githubusercontent.com/58999917/190092240-e66b692e-a141-491f-a6d5-83e1a4226a94.png)

- Find `Image and shape`. Click `Edit`.

![image](https://user-images.githubusercontent.com/58999917/190092388-09b68df0-6004-4fc8-b6d8-983910047a2c.png)

- Select the `Image` which you want to use, for example `Ubuntu`.
- Click `Change shape`. Select `Ampere`. Set `Number of OCPUs` to `4` and `Amout of memory (GB)` to `24`. Then click `Select shape`.

![image](https://user-images.githubusercontent.com/58999917/190092492-22e6a760-c9b8-4639-9884-d28f1ddba9fa.png)
![image](https://user-images.githubusercontent.com/58999917/190092584-fc89e694-78dc-436d-bd70-413017d3b69e.png)

- Scroll down to `Networking`. Click `Edit`. Select `Do not assign a public IPv4 address`.

![image](https://user-images.githubusercontent.com/58999917/190092680-95547bc3-c842-4b35-af8b-39b86aaa3bc2.png)

- Scroll down to `Add SSH keys`. Click `Save Private Key` and `Save Public Key`. Store them to a safe place. We’ll need it later.

![image](https://user-images.githubusercontent.com/58999917/190092746-7198d909-f961-44d2-8d1f-eac4278599d5.png)

- Before creating VM, press `F12` to open DevTools. Select tab `Network`.
- Click `Create`. You’ll receive an error.

```
Out of capacity for shape VM.Standard.A1.Flex in availability domain OFDj:AP-SINGAPORE-1-AD-1. If you specified a fault domain, try creating the instance without specifying a fault domain. If that doesn’t work, please try again later. Learn more about host capacity.
```
![image](https://user-images.githubusercontent.com/58999917/190092910-06fcae7d-0711-4bff-8ea9-32e56730d81b.png)

- In the DevTools, tab `Network`, search now for `instances` request. Right click on it. Select `Copy –> Copy as cURL (cmd)`.

![image](https://user-images.githubusercontent.com/58999917/190093000-0dd7cc51-4cae-44af-93de-bf4449cc929a.png)

- Copy the content to an editor. Find the part `–data-raw`.
- Open the `oci_auto.py` which you downloaded before and replace the values with the values in `–data-raw`.

```
instance_display_name = 'instance-********'
compartment_id = 'ocid1.tenancy.oc1..********'
domain = "OFDj:AP-SINGAPORE-1-AD-1"  # availability_domain
image_id = "ocid1.image.oc1.ap-singapore-1.********"
subnet_id = 'ocid1.subnet.oc1.ap-singapore-1.********'
ssh_key = "ssh-rsa ******** ssh-key-2022-01-14"
```

- The setup is now finished. You can run the script with the following commands.

```
$ pip install requests
$ pip install oci
$ python3 oci_auto.py
```

- If you see this, that means the script has run successfully.

![image](https://user-images.githubusercontent.com/58999917/190093325-fe5e0bd5-de68-4332-8d81-6026804b2875.png)

- It may take days until there is a free slot for you. The script will quit and give back a message when he creates a VM successfully.

![image](https://user-images.githubusercontent.com/58999917/190093378-fa8ab512-340d-429b-9dfa-e2b62c1c64ad.png)

- After the VM gets created, go to `Instances`, select the newly created instance.
- Scroll down to `Resources`. Select `Attached VNICs`. Click on the newly create VNIC.
- Scroll down to `Resources`. Select `IPv4 Addresses`. `Edit` and select `Ephemeral public IP`. Click Update.

![image](https://user-images.githubusercontent.com/58999917/190093652-9a33012a-f425-4744-9a57-7625d2516d4d.png)

- Open `PuTTY Key Generator`, `Load` the private key you saved before. Click `Save private key` to generate a key format (*.ppk) which can be used by Putty. Save it to a safe place.
- Open `PuTTY`, go to `Connection –> SSH –> Auth`, select the private key generated by `PuTTY Key Generator` before.

![image](https://user-images.githubusercontent.com/58999917/190093895-f2467b3f-3f6d-444e-86ec-44587ec42f07.png)

- In `PuTTY`, go to `Session`, enter the IP you get from instance details. Then `Open` the connection. You’ll be asked for username which is also available in section of instance details.

![image](https://user-images.githubusercontent.com/58999917/190094026-bfc2499f-3dd5-41bf-b00a-0dc2d259a8c4.png)

