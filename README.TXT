Overview
--------
bgitsh (Beej Git Shell--not a real shell) is yet another git access
control mechanism to allow repo-level access control to different users
in a single account.

It does this by using the command= option for ssh to run bgitsh when an
authorized user (by RSA key) ssh's into the single git account.  bgitsh
checks the user name against data in an access control file, and, if
access is granted, executes the requested git command.

When To Use
-----------
Never.

There are probably better tools to do what you want.  I wrote this as a
learning exercise.  Use gitosis or gitolite instead.

That being said, this might fit the bill for cases where you want a
simpler install.


Security
--------
A modest effort has been made to make this a secure app.  No guarantees,
of course.


Admin Setup
-----------
0. Install bgitsh (suggested: in ~/.ssh/) on the git host, logged in
   as the main git user.  Make it executable.

1. Get the client user's public RSA key.

2. Decide upon the client user's bgitsh username (can be anything, e.g.
   their email)

3. Add the user's public key to ~/.ssh/authorized_keys, passing their
   bgitsh username to bgitsh (the following is all on one line):

   command="~/.ssh/bgitsh user@example.com",no-port-forwarding,
     no-X11-forwarding,no-agent-forwarding
     ssh-rsa AAAAB[... giant public key ...]u+lF5S8L user@example.com

   (The ssh-rsa comment and bgitsh user name do not need to be the
   same, even though they happen to be in the above example.)

4. Add the user's repo permissions to ~/.ssh/bgitsh_access, using
   their bgitsh username.

   See examples/bgitsh_access for examples.


User Setup
----------
1. Make an RSA key for SSH:

      cd ~/.ssh
      ssh-keygen -t rsa -C you@example.com

   Name the key something else if you don't want to use the default
   (or if you already have a default key that you don't want to
   overwrite.)

2. Give your public key (the .pub) to the git admin.

3. From the admin, get the git username and hostname.

4. Edit your ~/.ssh/config file (create it if it doesn't exist).  Add
   this block for convenience:

   Host githost.example.com
       User gituser
       HostName githost.example.com
       IdentityFile ~/.ssh/id_rsa.git


Example bgitsh_access file
--------------------------

# bgitsh access file
#
# Format:
# 
# username reponame,perms [reponame,perms ...]
#
# Repo names can be regular expressions.
#
# Permissions are:
#
#   r   pull access
#   w   push access
#   rw  pull and push access
#   z   no access
#
# Repo names are tested in left to right order until the first match is
# made.  If no rules match, no access permission is granted.
#
# Examples:
#
# Allow read on repo1, read/write on repo2:
#
#   joeuser repo1,r repo2,rw
#
# Allow read on all repos, except the secret repo:
#
#   joeuser secret_repo,z .*,r
#
# Allow full access to all repos named "clientX_something", and nothing else:
#
#   joeuser clientX_.*,rw
#

