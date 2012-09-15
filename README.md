Tank Auth w/ Role-Based Access Control (RBAC)
==============================================

> **Undergoing beta testing**. This is currently being swept for bugs. I'll be running more tests and adding more features as I see fit along the way. Any bug testers will be much appreciated. Thanks!

This fork implements a **Role-Based Access Control** method popular in multiuser sites. This new release is bursting with so much vitamins and minderals it's practically it's own food group! Okay, maybe not.

New Features
------------
1. Role-Based Access Control for multi-user sites. For more info on RBAC, refer to [this post](http://www.tonymarston.net/php-mysql/role-based-access-control.html 'Role-Based Access Control').
1. Add custom fields to your registration page
1. You can approve users manually. They can register and activate but not enter until they are approved. Default value: `$config['acct_approval'] = TRUE`.
1. Replaced the default captcha with [Cool Captcha](http://code.google.com/p/cool-php-captcha/)

To do:
------
1. **Completed** - *Replace default captcha with [Cool Captcha](http://code.google.com/p/cool-php-captcha/)*
1. **Completed** - *Add custom fields to the registration page*
1. **Beta testing in progress** - *Role-Based Access Control (RBAC) w/ user-level overrides*
1. Buy myself an ice cream

Methods & Configurations
------------------------
### Role-Based Access Control

All arguments for the following methods can be found by searching through the *Tank_auth* library and *Users* model. Lift those sticky fingers and *Ctrl-F* like your project depended on it!

- `permit()`: The most important method of all since it checks if the user can do a certain thing (e.g. view a certain page)
- `add_permission()` and `remove_permission()`: Add/Remove permissions of roles. All permissions are listed in the `permissions.permission` table.
- `new_permission()`, `clear_permission()`, and `save_permission()`: Insert data in the `permissions` table.
- `add_override()`, `remove_override()`, and `flip_override()`: Override permissions on a per-user basis. This allows you to give/take away permissions to users outside of their active role/s.
- `add_role()`, `remove_role()`, `change_role()`: Basic role management for *Users*. If you're looking to create new roles in the `roles` table, I suggest you do it in SQL.

More methods may have been added but those above are the ones you'll most likely be using.

### Custom Registration Fields

Before you add any custom registration fields, make sure you've created a field for it in the `user_profiles` table. That's where your user's data will be saved. Do this and the sky won't fall on your head.

Assign custom fields using the `$config['registration_fields']` array.

> **How it works:** Field data is serialized in the `users.meta` field until the account is activated. After that, all data is transferred to the `profiles` table where it stays for the duration of the account.

**1. Text input format:** *`array(Name, Label, Rules, 'text', Attributes)`*

How to use:

	// Sample 1
	$config['registration_fields'][] = array('name', 'Full name', 'trim|required', 'text');
	
	// Sample 2 w/ optional attribute
	$attr = array('class'=>'bigfont');
	$config['registration_fields'][] = array('name', 'Full name', 'trim|required', 'text', $attr);

**2. Radio format:** *`array(Name, Label, Rules, 'radio', Selection, Opening tag, Closing tag)`*

How to use:

	$selection = array(
		'm'=>'Male',
		'f'=>'Female'
	);
	
	// Sample 1
	$config['registration_fields'][] = array('gender', 'Gender', 'trim|required|alpha|max_length[1]', 'radio', $selection);
	
	// Sample 2 w/ <p> tags separating each radio item
	$config['registration_fields'][] = array('gender', 'Gender', 'trim|required|alpha|max_length[1]', 'radio', $selection, '<p>', '</p>');

Radio fields do not have any assigned default value (unlike checkbox fields).


**3. Dropdown format:** *`array(Name, Label, Rules, 'dropdown', Selection)`*

How to use:

	// Make the initial value '0' so the field will fail if selected
	$selection = array(
		'0'=>'- choose -',
		'US'=>'USA',
		'PH'=>'Philippines',
		'JP'=>'Japan'
	);
	$config['registration_fields'][] = array('country', 'Country', 'trim|required|callback__not_zero', 'dropdown', $selection);

The callback `_not_zero` simply returns `FALSE` if the value is `0`.

**4. Checkbox format:** *`array(Name, Label, Rules, 'checkbox', Checkbox Label, Default Checked)`*

How to use:

	// Sample 1
	$config['registration_fields'][] = array('money', 'Money', 'trim|numeric', 'checkbox', 'I want money');
	
	// Sample 2: checkbox is checked by default
	$config['registration_fields'][] = array('money', 'Money', 'trim|numeric', 'checkbox', 'I want money', TRUE);

How to use Tank Auth w/ RBAC
------------------------------
_Editor's note:_ Here is a complete description on how to use this fork. I tried to be as thorough as possible but if there's anything unclear then let me know and I'll clear it up for you.
### I. Empty all data
1. **Clear all table data: `roles`, `permissions`, and `role_permissions`.** These are values I used for testing and are simply examples on how to populate these 4 fields. You may opt to keep them to familiarize yourself of its use.

> You may choose to retain the data in the `roles` table. These include the most common roles found in every system and can be of use in your project.

### II. Populate the `permissions` and `roles` table
1. **Populate the `permissions` table.** These list down the permissions your users can take. Keep values for the `permission` field short and it's recommended that the first word be a verb (e.g. 'create user').
1. **Fields in `permissions` table: `description`, `parent`, and `sort`.** These are _recommended_ but are optional since they are used if you are displaying your options in HTML in case you allow certain roles to manage permissions through the use of a form (e.g. _Admin_ role). If you prefer to use SQL instead of creating a form then these can be left empty.
1. **Populate the `roles` table: create the roles your project will use.** This RBAC setup allows users to have _1 or more roles_. This prevents an influx of creating too many roles to accomodate user responsibilities.
1. **Fields in `roles` table: `role`, `full`, and `default`.** Keep the `role` field one-word and lowercase for easy typing since this is what you will use when running the `permit()` method. The `full` field is the full name of the role and can be used if you're displaying your options in HTML in case you allow certain roles to manage permissions through the use of a form (e.g. _Admin_ role). The `default` field specifies which role is the default role when users sign up.
1. **Populate the `role_permissions` table.** Match which permissions belong to which roles.

> Permissions are assigned to roles and not users. For special cases where a certain user needs a permission outside of their role, use the `overrides` table (Part IV).

### III. Assigning roles to users
1. **Just assign the default role in the `roles.default` field and the system does the rest.** When a user signs up, he/she will be given that role.
1. If the user is the very first user ever, they will automatically be given a role with the _role\_id_ of `1`. This is preferrably the _Admin_ role. This action bypasses the default role.

> Roles are only given once a user has been activated and not before. This keeps your tables clean and free from any unnecessary inserts. Users are only given 1 role upon registration but if you want to add more roles to a user, you'll have to do it _after_ they're activated using the `add_role()` method to do so.

### IV. Give user extra permissions outside of their role (Optional)
_Editor's note:_ This is used on a case-to-case basis only. It allows for extended flexibility on how you manage your users.

1. **Customize your user's permissions.** If you want to give certain user's extra power outside of their role, you can use the `overrides` table for that. This table allows you to add permissions outside of a user's role as well as remove permissions already within their role.
1. **Fields in `overrides` table: `allow`.** This says whether a user can or cannot do a certain permission. Use `add_override()` to add to a user's permissions. The `flip_override()` method flips the `allow` field from a `1` to a `0` and vice versa.

> When using the `permit()` method, this checks your existing permissions from your role/s as well as any overrides you may have on certain permissions. Cool!

Changelog
---------
1. Wrote the _How to use Tank Auth w/ RBAC_ directives (whew!). - *Sep 15, 2012*
1. Initial RBAC functionality achieved: Add/Remove/Change roles - *Sep 12, 2012*
1. Check permissions with the `permit()` method which allows you to override permissions on a user-level scope - *Sep 11, 2012*
1. Add custom fields to the registration page - *Sep 10, 2012*
1. Implement Cool Captcha instead of the default captcha - *Aug 29, 2012*
