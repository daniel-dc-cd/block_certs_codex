# block_certs_codex

## Intent

This guide is a work in progress and should not be considered a completed guide, however, the goal of this document is to be a complete guide to creating, issuing and viewing Blockcert certificates. As the Blockcerts technology changes so will, hopefully, this guide.

## Guide start

First clone the cert-tools package from [here](https://github.com/blockchain-certificates/cert-tools). 

A few things are going to occur in this package, first we're going to modify some settings to create a custom template, then we're going to use that template in conjunction with our roster.csv file to generate some unsigned certificates which we will send to our cert-issuer.

Follow the installation procedure outlined in the readme of the link from above.

### Overview

> See sample_data for example configuration and output. conf-mainnet.ini was used to create a batch of 2 unsigned certificates on the Bitcoin blockchain.

> The steps are:

+ Create the template
    + Update the config file to contain the correct data to populate the certificates
    + Place the needed images in images/ and point to them in the config file

+ Run create_certificate_template.py, which resulted in the certificate template /certificate_templates/test.json
    + Instantiate the batch
    + Add the recipient roster (in this case rosters/roster_testnet.csv) with the recipient's Bitcoin addresses.
    + Run 'instantiate_certificate_batch.py', which resulted in the files in unsigned_certificates

+ Then the unsigned certificates were copied to cert-issuer for signing and issuing on the blockchain.

### 1 - Creating the template

### Changing Images

To change the logo image and the signature image you will need to add the image files to the images folder inside sample_data and then make some adjustments to the conf.ini file. Look for the #images tag and there you can replace the default images file names with the names of the files you just uploaded. 

### Changing the text on the certificate.

When a user goes to view the certificate they are going to see text from the certificate as well as some text that is generated by the cert-viewer. The issuer and certificate information fields can be replaced with custom data on the conf.ini while some of the other text and images should be changed in the cert-viewer code.

The issuer_public_key=ecdsa-koblitz-pubkey: is generated from the bitcoin wallet you will use when switching from "regtest" to "testnet" or "mainnet" coins. For now, just use the defualt values.

### Adding a Custom field (WIP)
By incorporating your own JSON-LD inforamtion you'll be able to add to the context of the certificate. After adding some inforation to both your roster.csv and the conf.ini file you will still need to modify the cert-viewer if you want the custom field to render in the html.

First, add this line to the end of your conf.ini file

 `additional_per_recipient_fields = {"fields": [{"path": "$.xyz_custom_field","value": "*|XYZ_PLACEHOLDER|*","csv_column": "xyz_custom_field"}]}`

This will create the field `$.xyz_custom_field` in the larger context of the JSON-LD that will later get merged with the information from multiple sources. The `value` is the merge tag which is used as a placeholder in the template until information from the csv file can get incorporated. The `csv_column` is the name of the column in the roster.csv file which will provide the data for each recipient.

### Changing the roster
By default the file create_v2_template.py file will use the conf.ini settings and pull This file is located in the sample_data folder and you should edit this to add more or less recipients.

Now add the `xyz_custom_field` to the roster_testnet.csv file. Your file should look something like this

```
name,pubkey,identity,xyz_custom_field
Eularia Landroth,ecdsa-koblitz-pubkey:mtr98kany9G1XYNU74pRnfBQmaCg2FZLmc,eularia@landroth.org,xyz custom info 1!
Mcallister Greenborough,ecdsa-koblitz-pubkey:mkwntSiQmc14H65YxwckLenxY3DsEpvFbe,mcallister@greenborough.org,xyz custom info 1!
```

Next up we're going to modify the `create_v2_template.py` file. Look around like 91 and change to this:

```
    assertion = {
        '@context': [
            OPEN_BADGES_V2_CONTEXT, 
            BLOCKCERTS_V2_CONTEXT,
            {
                "xyz_custom_field" : {
                "@id": "http://schema.org/ComputerLanguage"
                }
            }
            
        ],
```

I'm still learning JSON-LD but what I think is going on here is we're adding our own custom field named "xyz_custom_field" and changing the default JSON-LD context to include our custom field and make it shorthand for the the link we included in the `@id` field. How the schema integrates with the roster csv file is still kind of a mystery to me so if you know please share!

### Creating the template!

Now that we've changed some text, modified some images, and added a custom field into we can generate a template. 

Use this command from the **cert-tools** directory.

`python cert_tools/create_v2_certificate_template.py -c conf.ini` 

If there aren't any errors you should have created a template and you can find the file `test_template.json` in the sample_data/certificates folder. You can open it up and inspect it to see if your customized information was added before you move on.

### 2 - Instantiating a batch

### Overview
Here we're going to use the information stored in the template and the roster file to create some unsigned certificates, these will be JSON files that we'll copy into our quick start Docker imaage to issue the certificates.

Once your roster_testnet.csv is populated with information to your liking, run this command.

`python cert_tools/instantiate_v2_certificate_batch.py -c conf.ini`

If all goes well, you should see that some files have appeared in the unsigned certificates folder. There should be as many json files are there are recipients.

These files should now be ready to be copied to the cert-issuer.

### 3 - Setting up the cert-issuer

Follow the instructions provided on the readme [here](https://github.com/blockchain-certificates/cert-issuer)

## Overview

If you want to use the custom templated certificates we just generated please read the helpful tip below first.

### Helpful tips!

You'll be copying your unsigned certificates you made in the cert-tools into the Docker container where they will be associated in a batch with a mock bitcoin transaction, the resulting files will then need to be transfered out of the Docker instance into the cert-viewer.

The command you'll want to use will look something like:

`docker cp <cert-tools>/sample_data/unsigned_certificates/. <docker-container-name>:/etc/cert-issuer/data/unsigned_certificates/ `

Once moved, continue on with the the fake bitcoin genereation and issuing.

---

If you encounter an issue where even after sending to address the issuer balance is lacking, take the issuer hash in the error message and interpolate that into this command

`bitcoin-cli sendtoaddress <issuer-hash-goes here> 5`

### 4 - Viewing the certificate

## TIPS
If you're developing on OSX when you issue a certificate you might encounter a SSL error that prevents you from accessing the balance of your wallet, check this website to update your certificates https://stackoverflow.com/questions/42098126/mac-osx-python-ssl-sslerror-ssl-certificate-verify-failed-certificate-verify

If pysha3 fails in build wheel, sudo apt-get install build-essential python-dev python3-dev

## Certificate tooling, issuing and viewing on Dojo Servers.

## cert issuing and viewer guide

```
    ############################################################################
    # Developer note                                                          #
    # This conf_***.ini files in both the issuer and the tools will need to   # 
    # be updated to include the new mainnet issuing address and private key   # 
    # when the mainnet wallet goes                                            #
    # live.                                                                   #
    #                                                                         #
    # If something about the certificates need to be updated, the certificate # 
    # template needs to be remade, make sure we test the certificates on our  # 
    # test-net dev server to make sure that the certificates will be created  # 
    # and validated prior to issuing certificates on main-net and burning     # 
    # REAL money.                                                             #
    ############################################################################
```

1. Move old graduate information to another sheet.

2. Put in information for new batch, fill out their, Name, a public key you get from [bitaddress testet](https://www.bitaddress.org/testnet=true) or [bitaddress](https://www.bitaddress.org) for real, (give the private key to the student, you can also use the batch creator to create multiple addresses at once), belt levels. IMPORTANT: this information is written directly into the certificate so make sure things look the way you want them to.

(!!NOTE!!) If there have been unsigned certificates left behind in the unsigned_certificates folder in the cert_tools_deployed data directory, remove the old files, we should't need to save these as we will only care about the blockchained certificates.

3. Navigate into the cert_tools_deployed and instatiate the batch of unsigned certificates by using this command on the cert tools server, activate the ct_venv virtual environment

    ```
    # Activate virtual environment
    source /home/ubuntu/cert_tools_deployed/ct-venv/bin/activate
    # Instantite unsigned certificate batch
    python cert_tools/instantiate_v2_certificate_batch.py -c conf.ini
    ```

4. After the batch of unsigned certificates are made, the files are save into the unsigned_certificates folder.

    ```
    /home/ubuntu/cert_tools_deployed/data/unsigned_certificates
    ```

5. Use this command to copy the unsigned certificate from cert-tools to the cert-issuer

    ```
    cp ~/cert_tools_deployed/data/unsigned_certificates/* ~/cert_issuer_deployed/data/unsigned_certificates/
    ```
6. Use these commands to issue the certificates, if the balance is insufficient contact certificates@codingdojo.com for add more bitcoin.

    ```
     # Activate virtual environment
    source /home/ubuntu/cert_issuer_deployed/ci_venv/bin/activate
    # Issue certificates
    python cert_issuer/issue_certificates.py -c conf_testnet.ini


7. Using filezila copy these the files to your local machine, then using filezilla again upload these unsigned certificates to the certificates_prod server directory 

    ```
    /home/ubuntu/cert-viewer/cert_data
    ```

8. Run this command

    ```
    $ service_restart
    ```

9. Go to certificate.dojo.new and review a few of the certificates that were recently updated and make sure the validations are working. If there are any issues, delete the offending certificates so they do no appear on the website.

## THINGS TO CHANGE FROM TESTNET TO MAIN NET
1 . cert-tools conf.ini
        issuer_public_key=ecdsa-koblitz-pubkey:<put new key here>
        regenerate issuer and revocation with new pubkey
2. cert-issuer conf_mainnet.ini
    change issuing address
    change bitcoin chain type
    put private key in pk_issuer.txt
3. cert-viewer
    put in new issuer.json 
    put in new revocation.json




