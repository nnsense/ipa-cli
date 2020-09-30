# ipa-cli

This script is to help with some common IPA management tasks, such as user, OTP and DNS management. It relies on the python freeipa module, see the [project docs](https://python-freeipa.readthedocs.io/en/latest/) for more info.
It is strongly advised to create a kerberos ticket with `kinit username`, `-u` and `-p` are provided to pass administrative username and password but the sole use of this is to automate the script, with plain username/password (no OTP).

This script requires 3 positional arguments: **service task and target**.
`service` is the IPA object to work with, can currently be dns, user, otp and host.
`task`, is the action to perform on the service, can currently be create, delete, disable, cleanup and find.
`target` is any attribute to search for, or a specific username to create, delete or disable. For DNS is a record name.


## FIND

Search all users by name, cn, uid. The returned object is listing all attributes from `user_find` plus last login and last failed authentication from ipa `user_status`.
By default, it returns all the attributes, you can limit to specific attributes by using `--filter` (`-f`). Attributes available are listed by searching without a filter. The result by using `--filter` will be a table. If `--filter` is set, the uid attribute is automatically added to the list.
By default, last login attribute is disabled on IPA, "for performance reasons". See [this webpage](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/enabling-tracking-of-last-successful-kerberos-authentication)  to enable it or you'll get an empty field.

Search a user called "dbowie", list all its attributes:
```
ipa-cli user find "dbowie"
```
Search a user called "dbowie" and show only some attribute. `--filter` (`-f`) accepts any IPA attributes, plus the custom attributes `lastlogin` and `lastfailedauth`.
```
ipa-cli user find "bowie" -f sn lastlogin lastfailedauth
```
Search for a user's assigned OTP. Like with user, use `--filter` (`-f`) as in the previous example to show only attributes:
```
ipa-cli otp find "dbowie"
```
Search for a DNS record: search can be by hostname or IP
```
ipa-cli dns find "172.31."
ipa-cli dns find "172.31." -f dn
```

#### --cleanup (switch available using --find, NOT WORKING with --filter)

Clean up objects found with --find using the following rules:
user - Search and DELETE accounts of which the last successful auth. Any OTP linked to the user is also deleted. Use --disable to disable instead of deleting the user
host - Search and DELETE hosts.
dns - Search and DELETE DNS entries. This is using the `--del-all` switch, which deletes both direct and reverse entries matching the search.
otp - Search and DELETE OTPs

```
ipa-cli user find "bowie" --cleanup
ipa-cli user find "bowie" --cleanup --disable
ipa-cli user find "172.31" --cleanup
```

Being based on "find", find's output can be used first to check what clean up will delete/disable.


**CREATE**

Create a new user. --password is optional, see the config file to set this field (default: 'ChangeMeSoon')
A group the user will be member of can be defined with --group (-g). OTP creation is mandatory, the uri of the user's OTP
will be displayed at the end of the creation, copy and paste the secret from the URI provided to show the QR code for your auth app.
```
> ipa-cli user create 'dbowie' -n 'David' -s 'Bowie' -g 'prod-users' 'more-groups'
```


**DELETE**

To delete a user, the uid must be provided. Note that the OTP belonging to the user is also deleted if exists.
```
ipa-cli user delete "dbowie"
ipa-cli otp delete "dbowie"
```

