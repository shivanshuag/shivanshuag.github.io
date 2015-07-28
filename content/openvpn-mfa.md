+++
Description = ""
Tags = ["OpenVPN", "Mozilla"]
date = "2015-07-10T13:35:54+05:30"
menu = "main"
title = "Multi-Factor Authentication In OpenVPN"
+++

{{< figure src="/images/MWoS.png" class="pull-right" >}}

This project was done as a part of [Mozilla Winter of Security 2014](https://blog.mozilla.org/security/2014/05/15/introducing-mozilla-winter-of-security-2014/) programme.
The objective of this project is to implement native Multifactor Authentication Support for OpenVPN. The existing support for Multifactor Authentication in OpenVPN is through deferred C plugins and pythons scripts which has lots of limitations due to the nature they are implemented.


## Need to Native MFA support

The plugins are present on the server side where they are called at appropriate places in the authentication process of OpenVPN with credentials. But on the client side, openVPN provides the users options for authenticating through static keys, username/Password or certificates. There is no support for entering OTP or any other credential here. This user ahs to resort to using hacky ways to circumvent this limitation like entering the OTP cancatenated with password which can lead to even more limitations on the length of password and so on. An example plugin, used by mozilla, can be found at https://github.com/mozilla-it/duo_openvpn .

This method can also limits the type of credentials that can be taken from the user. Typical Multifactor Authentication solutions also provide Push notifications on smartphones as a way for authentication along with the regular OTP method. Native Support for Multifactor Authentication in OpenVPN will help in supporting these modes of authentication.

Another use of native MFA support can be maintaining user sessions so that the user does not have to enter the second factor of authentication every time he/she logs in but say only once in a month. These reasons demand for native MFA suport in OpenVPN.

## MFA Implementation

Our scheme of implementation provides support for three different types of authentication: OTP, Push and User-pass (as of yet) in addition to the certificate check that happens as a part of the normal TLS handshake. 

We have modified the [key-method 2](https://openvpn.net/index.php/open-source/documentation/security-overview.html) packet structure used for authentication to include MFA credentials. MFA username and password are sent to the script/plugin in a manner similar to the existing auth-user-pass soution . In case of OTP when there is no username, CN of the user is sent as username. In case of PUSH, CN is sent as username and an empty string as password. Three types of plugins are supported for MFA, one for each of push, otp and user-pass.

The type for push is - `OPENVPN_PLUGIN_AUTH_MFA_PUSH_VERIFY`

for otp - `OPENVPN_PLUGIN_AUTH_MFA_OTP_VERIFY`

for user-pass - `OPENVPN_PLUGIN_AUTH_MFA_USER_PASS_VERIFY`

## MFA Usage and Config

In the server config, put the supported authentication methods as follows

{{< highlight bash >}}
mfa-method method-type [script_file via-env|via-file]
{{< /highlight >}}
Here method type can be 'otp', 'push', 'user-pass'. For example

{{< highlight bash >}}
mfa-method otp auth.pl via-file
{{< /highlight >}}

The three methods differ in what credentials are asked from the user for authentication. In otp, only password is asked. In push, nothing is asked. This is meant for authentication by Push notification on mobile phones which can be handled by a deferred auth plugin on the server side. In user-pass, both username and password are asked from the user. If you want to use a plugin for authentication on the server, include the following lines in the config

{{< highlight bash >}}
mfa-method method-type
plugin plugin_shared_object_file
{{< /highlight >}}

In the client config, put the following line

{{< highlight bash >}}
mfa-method mfa-type
{{< /highlight >}}

User can authenticate using one method only. Multiple mfa-method lines will give an error. 

## Session Support Implementation

On startup the OpenVPN server generates a random key (48 bytes). When a client successfully authenticates with MFA enabled, the server generates a session token using the tls1_PRF function with the key as the secret and (Client CN + Expiry timestamp) as the data. The expiry timestamp and the token are sent to the client. When the client wants to connect later, it must send the token and the expiry timestamp. If verification succeeds, MFA auth is bypassed. If not, auth fails and the client is prompted for MFA authentication if the auth-retry directive is set to "interact".

## Session Support Usage and Config

The server needs to provide the following configuration option:

{{< highlight bash >}}
mfa-session-expiration session-validity
{{< /highlight >}}

`session-validity` is the duration (in hours) for which the session cookie is valid.

The client needs to provide the following configuration option:

{{< highlight bash >}}
mfa-session-file filename
{{< /highlight >}}

`filename` is the path to the file in which session tokens will be stored. If the filename is not provided, then we warn the user and disable session-support.

## Code, Demo and Presentations

The code for this project can be found at [https://github.com/mozilla/openvpn](https://github.com/mozilla/openvpn). The project wiki can be found [here](https://wiki.mozilla.org/Security/Mentorships/MWoS/2014/OpenVPN_MFA).

A video where we have explained the project and given a demo is present [here](https://air.mozilla.org/mwos-2014-openvpn-mfa/).
