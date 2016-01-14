http://www.ejabberd.im/mod_filter

mod_filter
==========

Flexible filtering by server policy

This module allows the admin to specify packet filtering rules using ACL and ACCESS.

## Install


Add the module to the list of modules on ejabberd.cfg:
```erlang
{modules, [
  ...
  {mod_filter, []},
  ...
]}.
```
Add to ejabberd.cfg the default ACCESS configuration:
```
{access, mod_filter, [{allow, all}]}.
{access, mod_filter_presence, [{allow, all}]}.
{access, mod_filter_message, [{allow, all}]}.
{access, mod_filter_iq, [{allow, all}]}.
```
Then modify those ACCESS rules to your needs. You can see examples below.
Recompile and restart ejabberd.

##Configuration examples

The configuration of rules is done using ejabberd's ACL and ACCESS, so you should also study the corresponding section on ejabberd guide. This are examples that may help you to understand how it works.

###Example 1
```
%% Admins can send anything.  Others are restricted in various ways.
{access, mod_filter, [{allow, admin},
 		      {restrict_local, local},
 		      {restrict_foreign, all}]}.

%% Local non-admin users can only send messages to other local users.
{access, restrict_local, [{allow, local},
			  {deny, all}]}.
%% Foreign users can only send messages to admins.
{access, restrict_foreign, [{allow, admin},
 			    {deny, all}]}.
```
###Example 2

On this example, the users of a private vhost (example3.org) can only chat with themselves, so that particular vhost will have no connection to the exterior. The other vhosts on the server are completely unrestricted. The administrators are also unrestricted.
```
% This ejabberd server has three virtual hosts
{hosts, ["example1.org", "example2.org", "example3.org"]}.

% This ACL will match any user or service (MUC, PubSub...) hosted on example3.org
{acl, ex3server, {server_glob, "*example3.net"}}.

% The main mod_filter rule allows any admin, but restricts example3 and the rest of packets
{access, mod_filter, [{allow, admin},
                      {restrict_ex3, ex3server},
                      {restrict_nonex3, all}]}.

% This rule, which applies to packets sent from Ex3 non-admin users,
% allows packets sent to Ex3 server (packets internal to the vhost) and denies anything else.
{access, restrict_ex3, [{allow, ex3server},
                        {deny, all}]}.

% This rule, which applies to the rest of packets (the ones that are not sent from Ex3),
% allows all packets to admins (allowing replies to stanzas from Ex3 admins),
% denies all other access to Ex3, and allows access to anything else.
{access, restrict_nonex3, [{allow, admin},
                           {deny, ex3server},
                           {allow, all}]}.
```
###Example 3
Allow just some MSN users (romeo and juliet) using the transport msn.example.com to comunicate with the users of the server.
```
{acl, good_msn_users, {user, "romeo%hotmail.com", "msn.example.com"}}.
{acl, good_msn_users, {user, "juliet%hotmail.com", "msn.example.com"}}.
{acl, good_msn_users, {user, "", "msn.example.com"}}.
{acl, msn_users, {server_glob, "msn*"}}.

{access, mod_filter, [
  % Filter incoming messages; allow only good messages
  {allow, good_msn_users},
  {deny, msn_users},
  % Filter the rest, including outgoing messages
  {filter_msn, all}
]}.

{access, filter_msn, [
  % Users can send messages to good MSN users
  {allow, good_msn_users},
  % but not to other MSN users
  {deny, msn_users},
  % All non-MSN traffic is allowed
  {allow, all}
]}.
```

###Example 4

This server has two virtual hosts, one is typical and the other has only anonymous users. The anonymous users cannot send or receive presence stanzas from outside their vhost.
```
{hosts, ["localhost", "anon.localhost"]}.

{auth_method, [internal]}.
{host_config, "anon.localhost", [{auth_method, anonymous},
                                 {anonymous_protocol, both}]}.

{acl, anon_user, {server_glob, "*anon.localhost"}}.

{access, mod_filter, [{allow, all}]}.

{access, mod_filter_presence, [{allow, admin},
                      {restrict_anon, anon_user},
                      {restrict_no_anon, all}]}.
{access, restrict_anon, [{allow, anon_user}, {deny, all}]}.
{access, restrict_no_anon, [{allow, admin}, {deny, anon_user}, {allow, all}]}.

{access, mod_filter_message, [{allow, all}]}.
{access, mod_filter_iq, [{allow, all}]}.
```
###Example 5

This server has three virtual hosts. The first and second are incommunicated between them. The admins do not have such restriction.
```
{hosts, ["domain1.localhost", "domain2.localhost", "domain3.localhost"]}.

{acl, domain1, {server_glob, "*domain1.localhost"}}.
{acl, domain2, {server_glob, "*domain2.localhost"}}.

{access, mod_filter, [{allow, admin},
                      {restrict_dom1, domain1},
                      {restrict_dom2, domain2},
                      {allow, all}]}.
{access, mod_filter_presence, [{allow, admin},
                      {restrict_dom1, domain1},
                      {restrict_dom2, domain2},
                      {allow, all}]}.
{access, mod_filter_message, [{allow, admin},
                      {restrict_dom1, domain1},
                      {restrict_dom2, domain2},
                      {allow, all}]}.
{access, mod_filter_iq, [{allow, admin},
                      {restrict_dom1, domain1},
                      {restrict_dom2, domain2},
                      {allow, all}]}.

{access, restrict_dom1, [{allow, domain1},
                        {deny, all}]}.
{access, restrict_dom2, [{allow, domain2},
                        {deny, all}]}.
```
