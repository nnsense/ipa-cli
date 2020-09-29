# ipa-cli

This script is to help with some common IPA management tasks, such as user, OTP and DNS management.
It is strongly advised to create a kerberos ticket with `kinit username`, `-u` and `-p` are provided to pass administrative username and password but the sole use of this is to automate the script, with plain username/password (no OTP).

This script requires 3 positional arguments: **service task and target**.
`service` is the IPA object to work with, can currently be dns, user, otp and host.
`task`, is the action to perform on the service, can currently be create, delete, disable, cleanup and find.
`target` is any attribute to search for, or a specific username to create, delete or disable. For DNS is a record name.


## FIND

Search all users by name, cn, uid. The returned object is listing all attributes from `user_find` plus last login and last failed authentication from ipa `user_status`.
By default, it returns all the attributes, you can limit to specific attributes by using `--filter` (`-f`). Attributes available are listed by searching without a filter. The result by using `--filter` will be a table. If `--filter` is set, the uid attribute is automatically added to the list.
By default last login attribute is disabled, "for performance reasons". See [this webpage](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/enabling-tracking-of-last-successful-kerberos-authentication)  to enable it or you'll get an empty field.

Examples:
Search a user called "dbowie", list all its attributes:
```
> ipa-cli user find "dbowie"
```
Search a user called "dbowie" and show only some attribute. `--filter` (`-f`) accepts any IPA aatributes, plus the custom attributes lastlogin and lastfailedauth.
```
> ipa-cli user find "bowie" -f sn lastlogin lastfailedauth
```

Search for a user's assigned OTP. Like with user, use `--filter` (`-f`) as in the previous example to show only attributes:
```
ipa-cli otp find "dbowie"
```

#### --cleanup (switch available using --find)

Clean up objects found with --find using the following rules:
user - Search and DISABLE accounts of which the last successful auth is older than 90 days (TO DO).
host - Search and DELETE hosts of which last successful password change is older than 90 days (TO DO).
dns - Search and DELETE unmatched DNS entries not present as hosts on IPA. This should be run after the host clean-up
otp - Search and DELETE any OTP not matching any user.
Target can be "\*" for a cpmplete cleanup, or a specific object group, ie to delete all users create in 2019:
```
> ipa-cli user find "2019" --cleanup
```

Being based on "find", find's output can be used first to check what cleanup will delete/disable.
By default, --cleanup *deletes* the object, with users you may want to disable first, to do that use --disable:
```
> ipa-cli user find "2019" --cleanup --disable
```


**CREATE**

Creaete a new user. --password is optional, see the config file to set this field (default: 'ChangeMeSoon')
A group the user will be member of can be defined with --group (-g). OTP creation is mandatory, the uri of the user's OTP
will be displayed at the end of the creation, copy and paste the provided uri into a browser to show th QR code.
```
> ipa-cli user create 'dbowie' -n 'David' -s 'Bowie' -g 'prod-users' 'more-groups'
```


**DELETE**

To delete a user, the uid must be provided. Note that the OTP belonging to the user is also deleted if exists.
```
> ipa-cli user delete "dbowie"
```

