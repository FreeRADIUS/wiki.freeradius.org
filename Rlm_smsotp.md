Smsotp (Short Message Service One Time Password) is a freeradius module written by Holger Wollf from Siemens which implements two-factor authentication using login/password and an otp (one time password) delivered via SMS: After the user logged in using the correct username and password, a SMS with an otp (one time password) is send to the user's phone. The user is required to enter the otp in order to successfully authenticate himeself.

# Architecture
Smsotp ships as a module which can be compiled and dynamically loaded. It connects to a unix domain socket where a daemon (smsotpd) listens. Holger Wollf did not provide his deamon,
so Thomas Glanzmann modified a perl POE (Perl Object Event) script to provide a smsotpd implementation. Smsotpd is contacted by the smsotp module after an user provided a valid username and password using PAP. It than sends the otp via SMS. Thomas Glanzmann used SIPGATE (a german VOIP provider) but any service can be used including a mobile phone by adopting the code in smsotpd. Afterwards the daemon is contacted again in order to verify that the provided otp is correct.

# Installation
The smsotp module is not installed by default, but can be compiled and installed by running the following commands:
    cd src/modules/rlm_smsotp
    chmod +x configure
    ./configure --prefix=/path/to/installation
    make
    make install

# Configuration
To active smsotp the Auth-Type must be set to smsotp for the user. This can be done for all users by adding the following line to the users file:

    DEFAULT Auth-Type := smsotp

Freeradius needs to know what to do with the Auth-Type smsotp so the file sites-enabled/default need to be modified to include the following in the sections authenticate and Authorize:


        authenticate {
                Auth-Type smsotp {
                        pap or ntlm_auth
                        smsotp
                }
        
                Auth-Type smsotp-reply {
                        smsotp
                }
        }
        
        Authorize {
                smsotp
        }

# Tripwires

* Smsotp can only be used with pap and ntlm_auth (which is pap with active directory backend) but it can not be used with CHAP, MSCHAPv1 and MSCHAPv2.