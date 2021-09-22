

# Fidelity 2FA with TOTP Authenticator Apps (without Symantac VIP)


Fidelity investments supports 2FA login with SMS and Symantec VIP (as of May 2021).  SMS based 2FA is know to be fairly insecure and Symantec VIP requires downloading another authenticator app.  I wanted the security from app based TOTP 2FA, but by using existing authenticator apps (e.g. Authy, Google Auth) already installed on my phone where I keep other passcodes.

**This guide will show you how to setup app based 2FA on your favorite TOTP authenticator app so you can login to Fidelity without ever having to install Symantec VIP.**


Fortunately, the Symantec VIP app uses standard security protcols and [some clever people](https://github.com/dlenski/python-vipaccess) have developed an opensource python client to replace the standard phone app.  You can use this python client to emulate the Symantec VIP app, extract the key, and install into your favorite TOTP app.  I have tested this with Yubio and Microsoft Authenticator apps only - but it should work fine with others apps like Google Authenticator and Authy.

This guide is for OSX but as long as you can install python, python-vipaccess, and qrencode on your OS it should still work.

**Setup Steps:**

1. Make sure you have `brew` `pip3` and `python3` installed in your terminal.  Brew is a package manager for OSX and pip is a package manager for python.
[Install brew](https://brew.sh/) first.  Once installed you can install python3 with `brew install python3` - This should also install pip3 at the same time.

2. Install [`python-vipaccess`](https://github.com/dlenski/python-vipaccess) with the python package manager:

		pip3 install python-vipaccess

3. Check you can run `python3 vipaccess` from the command line.  You should be able to find it in your python bin directory where it can be run directly (e.g. in `/Users/username/Library/Python/3.7/bin`)

4. Now vipaccess is installed, run the following command to create a new Symantec VIP ID.  This ID is like a secret key normally generated in the VIP app which is later communicated to Fidelity.

		python3 vipaccess provision -p -t VSMT

You should see the following somewhere in the output (obviously with a different credential number):

	You will need the ID to register this credential: VSMT1234516

Make a note of that credential you will need it later.

5. In the vipaccess output you will also see something like this:

		otpauth://totp/VIP%20Access:VSMT123456?secret=7YLUYWBBNQZEQX626M23PV6TE3BV4FMD&digits=6&algorithm=SHA1&issuer=Symantec&period=30

Copy the text and modify the `VIP%20Access:VSMT123456` and `issuer=Symantec` parts so it looks like this:

<pre>
otpauth://totp/<b>your_fidelity_username</b>?secret=B62TJYTHYEO5GEIHEODYHY77HFUK6ZEI&digits=6&algorithm=SHA1&<b>issuer=Fidelity</b>&period=30
</pre>

This step is purely aesthetic to make sure it's labled correctly in your TOTP app.


6. Install `qrencode` so you can generate a QR code.  You could probably enter the final otpauth URL in step 5 directly in your TOTP app, but I have not tried that.

		brew install qrencode


7. Run this command to generate a QR code.  Replace the last part of the command with your own otpauth URL from step 5:

		qrencode -o qr.png -s 15 "otpauth://totp/your_fidelity_username?secret=B62TJYTHYEO5GEIHEODYHY77HFUK6ZEI&digits=6&algorithm=SHA1&issuer=Fidelity&period=30"


8. Open the generated `qr.png` file and you should see a QR code.  Scan the QR code in your TOTP auth app of choice and you should see it added and passcodes get generated every 30 seconds.

9. Call Fidelity and ask to setup 2FA with VIP.  Tell them you already have the Symantec VIP app installed.  They will then ask you for your credential/ID, this is where you tell Fidelity the credential generated in step 4 (e.g. VSMT1234516 - yours will be different).  They will add the credential to your account and enable 2FA.

Fidelity should then ask you to try logging in while your still on the phone.  Enter username/pass as normal but now you should be asked to enter your 6 digit passcode - Enter the passcode from your TOTP authenticator app and it should log you in.

10. Delete the 'qr.png' file as this is sensitive data.


**Note to Fidelity:**
It's good that Fidelity support app based 2FA, but adoption could be increased by:
1. Offically support multiple TOTP apps (like Google Autenticator) with existing security standards.  Symantec VIP can still be supported but at least give your customers the option of using their existing passcode apps.
2. Create the key and QR code directly on the Fidelity website after login and reduce burden on your call center.  It's standard for this process to be automated so a call shouldn't be required (and maybe a security risk if a 2FA reset can be socially engineered over the phone).
3. Support hardware security key based login for example with a Yubikey or Duo.
