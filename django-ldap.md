#Error: Ldap connects with django but login is still unsuccessful
#Django version: 1.10.5 or 1.9.2

##Ldap connection is successful but user can't login to Django admin panel

(ldap == openldap)

As it turns out, connecting Django with Ldap is not as easy as it might seem.
I followed the [documentation](https://pythonhosted.org/django-auth-ldap/index.html) exactly but still could not authenticate.

Then Google told me that my user had to be a member in all 3 Django groups(active, staff, superuser).
So I did that to.
But still nothing.

I had logging enabled but the only thing shown there was "Populating Django user test" and nothing else. So it wasn't much of a help.

I found the issue when I decided to look at my sqlite database. 
Users were being stored correctly, with all their attributes etc. 

But there was one error.

Previously one of my users had logged in perfectly. But I had used the def shown below to make him a django staff member.
Obviously he could not edit anything so I just made him a superuser instead. And it worked. 

When I tried to do it a 2nd time, it failed, because the user no2 was a superuser but not a staff member. 

Turns out he has to be both a superuser member and a staff member.

So I added the defs below to my models.py and it all worked.

```
def make_super(sender, user, **kwargs): 
    user.is_superuser = True 

populate_user.connect(make_super)
```

```
def make_staff(sender, user, **kwargs):
    user.is_staff = True

populate_user.connect(make_staff)
```
(defs were not made by me so no credits for that, I just can't seem to be able to find the author)
