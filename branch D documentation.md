# IBSng JSON-RPC API Documentation

This documentation is converted from the `handlers_D.xml` definition file. It describes all available JSON-RPC methods of the IBSng system. The API listens on port **1237** by default.

---

## Common Parameters

These parameters can be included in any request (unless stated otherwise). They control authentication, session, date formatting, and remote IP logging.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `auth_type` | choice | Yes | Type of authentication: `ADMIN`, `NORMAL_USER`, `VOIP_USER`, `ANONYMOUS`, `MAIL` |
| `auth_name` | string | Yes | Username (or admin name) |
| `auth_pass` | string | Yes | Password |
| `auth_session` | string | No | Session ID returned by `login.login`. If provided and valid, you can omit `auth_type`, `auth_name`, `auth_pass`. |
| `auth_remoteaddr` | string | No | Remote IP of user or admin (important for audit logs) |
| `date_type` | choice | No | Date output format: `gregorian` (default), `jalali`, or `relative` |

---

## Handler: `admin`

Methods for managing administrators.

### `addNewAdmin`
- **Auth type:** `ADMIN`
- **Required permission:** `ADD NEW ADMIN`
- **Input (dict):**
  - `admin_username` (string)
  - `admin_password` (string)
  - `admin_isp_name` (string)
  - `name` (string)
  - `email` (string)
  - `comment` (string)
  - `admin_has_otp` (bool)
  - `admin_request_limit` (int)
- **Output:** `int` – Admin ID

### `getAdminInfo`
- **Auth type:** `ADMIN`
- **Input:** `admin_username` (string)
- **Output (dict):** admin details including `admin_id`, `username`, `name`, `email`, `comment`, `isp_name`, `creator_admin_username`, `locked`, `last_request_ip`, `last_activity` (epoch time), `online_status`, `admin_has_otp`.

### `getAdminISPID`
- **Auth type:** `ADMIN`
- **Input:** `admin_username` (string)
- **Output (dict):** `isp_id` (string)

### `changePassword`
- **Auth type:** `ADMIN`
- **Required permission:** `CHANGE ADMIN PASSWORD` (not needed if changing own password)
- **Input:** `admin_username` (string), `new_password` (string)
- **Output:** `null`

### `getAllAdminUsernames`
- **Auth type:** `ADMIN`
- **Required permission:** `SEE ADMIN INFO`
- **Two variants:**
  1. No input → returns list of all admin usernames (all ISPs)
  2. Input `isp_name` (string) → returns list for specific ISP
- **Output:** list of strings

### `getAllAdminInfos`
- **Auth type:** `ADMIN`
- **Required permission:** `SEE ADMIN INFO`
- **Added in tag 320**
- **Input:** optional `isp_name` (string)
- **Output:** list of dicts, each containing `admin_id`, `username`, `name`, `email`, `comment`, `isp_name`, `creator_admin_username`, `locked`, `last_request_ip`, `last_activity`, `online_status`, `admin_has_otp`.

### `updateAdminInfo`
- **Auth type:** `ADMIN`
- **Required permission:** `CHANGE ADMIN INFO`
- **Input (dict):** `admin_id` (int), `admin_username` (string), `admin_isp_name` (string), `admin_locked` (bool), `name` (string), `email` (string), `comment` (string), `admin_has_otp` (bool), `admin_request_limit` (int)
- **Output:** `null`

### `deleteAdmin`
- **Auth type:** `ADMIN`
- **Required permission:** `DELETE ADMIN`
- **Input:** `admin_username` (string)
- **Output:** `null`

### `generateOTP`
- **Auth type:** `ADMIN`
- **Required permission:** `CHANGE ADMIN PASSWORD`
- **Input:** `pass_len` (int), `pass_count` (int), `admin_username` (string)
- **Output:** list of strings (passwords)

### `searchOTPs`
- **Auth type:** `ADMIN`
- **Required permission:** `SEARCH OTP ADMINS`
- **Input (complex):**
  - `conds` dict with optional filters: `creation_date_from`, `creation_date_from_unit`, `creation_date_to`, `creation_date_to_unit`, `used_date_from`, `used_date_from_unit`, `used_date_to`, `used_date_to_unit`, `expired` (yes/no), `available` (yes/no)
  - `_from` (int), `to` (int) – pagination
  - `sort_by`: `otp_id`, `creation_date`, `used_date`
  - `desc` (bool)
- **Output (dict):** `total_rows` (int), `report` list of dicts with OTP details.

### `expireAllAdminOTPs`
- **Auth type:** `ADMIN`
- **Required permission:** `EXPIRE OTP ADMINS`
- **Input:** none
- **Output:** `null`

---

## Handler: `bw`

Bandwidth limit management (based on tag `C_lan_acc_staging_327`).

### `addInterface`
- **Auth type:** `ADMIN`
- **Required permission:** `CHANGE BANDWIDTH MANAGER`
- **Input:** `hostname` (str), `interface_name` (str), `comment` (str), `host_type` (`Linux` or `Mikrotik`)
- **Output:** `null`

### `addNode`
- **Input:** `hostname`, `interface_name`, `parent_id`, `rate_kbits`, `ceil_kbits`, `priority`
- **Output:** `null`

### `addLeaf`
- **Input:** `leaf_name`, `parent_id`, `default_rate_kbits`, `default_ceil_kbits`, `total_rate_kbits`, `total_ceil_kbits`, `default_priority`, `total_priority`
- **Output:** `null`

### `addLeafService`
- **Input:** `leaf_name`, `dst_ip`, `protocol` (tcp/udp/icmp/ip), `filter` (format: `sport COMMA_SEPARATED_PORTS` or `dport …`), `rate_kbits`, `ceil_kbits`, `priority`
- **Output:** `null`

### `getInterfaces`
- **Output:** dict with dynamic keys `"hostname/interface_name"` → value dict containing `interface_id`, `interface_name`, `hostname`, `host_type`, `comment`.

### `getNodeInfo`
- **Input:** `node_id`
- **Output:** dict with interface details, rates, priority, pkts, bytes, etc.

### `getLeafInfo`
- **Input:** `leaf_name`
- **Output:** dict with leaf details including `services` list.

### `getTree`
- **Input:** `hostname`, `interface_name`
- **Output:** list of length 3: node_id, child node subtrees (recursive list), list of child leaf names.

### `delLeafService`, `delNode`, `delLeaf`, `delInterface`
- **Input:** respective identifiers
- **Output:** `null`

### `getAllLeafNames`
- **Output:** list of leaf names (string)

### `updateInterface`, `updateNode`, `updateLeaf`, `updateLeafService`
- **Input:** respective fields to update
- **Output:** `null` (except `updateLeaf` returns a dict)

### `addBwStaticIP`, `updateBwStaticIP`, `delBwStaticIP`, `getAllBwStaticIPs`, `getBwStaticIPInfo`
- Manage static IP bandwidth mappings.

### `getActiveLeaves`
- **Input:** `order_by` (str), `desc` (bool)
- **Output:** list of lists containing IP, username, and send/receive leaf info.

### `getLeafCharges`
- **Input:** `leaf_name`
- **Output:** list of charge names that use this leaf.

### `getRealHostInterfaces`
- **Input:** `hostname`, `host_type` (`Linux`/`Mikrotik`)
- **Output:** list of interfaces installed on the host.

---

## Handler: `isp`

ISP (Internet Service Provider) management.

### `addNewISP`
- **Auth type:** `ADMIN`, **Permission:** `MANIPULATE ISP`
- **Input:** `isp_name`, `parent_isp_name`, `isp_has_deposit_limit` (bool), `isp_deposit` (float), `isp_mapped_user_id` (int or -1), `isp_auth_domain`, `isp_web_domain`, `isp_email`, `prevent_neg_deposit_login` (bool), `isp_comment`, plus removed field `isp_failed_user_id`.
- **Output:** `null`

### `updateISP`
- Similar input, adds `isp_id` and `isp_locked`.

### `getAllISPNames`
- **Output:** list of ISP names the admin has access to.

### `listISPsWithIDs` (tag 320)
- **Output:** list of `[isp_id, isp_name]`

### `getAllISPInfos` (tag 320)
- **Output:** list of dicts with full ISP info.

### `getISPInfo`
- **Input:** either `isp_name` or `isp_id`
- **Output:** full ISP info dict.

### `changeISPDeposit`
- **Input:** `isp_name`, `deposit_amount`, `comment`
- **Output:** `null`

### `deleteISP`
- **Input:** `isp_name`
- **Output:** `null`

### `getISPTree`
- **Input:** optional `isp_name` (defaults to admin’s owner ISP)
- **Output:** nested dict representing ISP tree.

### `getAllISPMappedUserIDs`
- **Permission:** `GOD`
- **Output:** list of mapped user IDs.

### Page style methods:
- `getISPPageStyle`, `setISPPageStyle`, `resetISPPageStyle` – manage custom logo/colors for ISP login pages.

### `getISPUsersCredit`
- **Output:** dict with ISP ID as key and total credit as value.

---

## Handler: `perm`

Admin permission management.

### `hasPerm`
- **Input:** `perm_name`, `admin_username`
- **Output:** bool

### `canDo`
- **Input:** `perm_name`, `admin_username`, `params` (list)
- **Output:** bool

### `getPermsOfAdmin`
- **Input:** `admin_username`
- **Output:** list of permission dicts (name, category, value_type, etc.)

### `getAllPerms`
- **Output:** list of all permission definitions.

### `getAdminPermVal`
- **Input:** `admin_username`, `perm_name`
- **Output:** dynamic (depends on permission type)

### `changePermission`
- **Permission:** `CHANGE ADMIN PERMISSIONS`
- **Input:** `admin_username`, `perm_name`, `perm_value`
- **Output:** `null`

### `delPermission`, `delPermissionValue`
- Remove whole permission or specific value.

### Template methods:
- `savePermsOfAdminToTemplate`, `getListOfPermTemplates`, `getPermsOfTemplate`, `loadPermTemplateToAdmin`, `deletePermTemplate`, `loadPermsFromAnotherAdmin`

---

## Handler: `charge`

Charge (service plan) management.

### `listCharges`
- **Input:** optional `isp_id` (comma-separated)
- **Output:** list of charge names.

### `getChargeInfo`
- **Input:** `charge_name`
- **Output:** dict with charge details and `charge_rules` list.

### `addNewCharge`
- **Permission:** `CHANGE CHARGE`
- **Input:** `charge_name`, `comment`, `isp_name`
- **Output:** `int` – Charge ID

### `copyCharge`
- Two variants: just one copy or specify `copy_count`
- **Output:** list of new Charge IDs.

### `updateCharge`, `deleteCharge`
- Standard update/delete.

### `addNewChargeRule`, `updateChargeRule`, `deleteChargeRule`
- Manage rules inside a charge.

### `updateChargeRuleAttrs`
- **Input:** `charge_rule_id`, `charge_name`, `update_attrs` (dict), `delete_attrs` (list)
- **Output:** `null`


## Handler: `extra_charge`

Extra charge profiles – rules that apply additional charges/credits at specific times (e.g., monthly, daily).

### `getExtraChargeProfileNames`
- **Auth type:** `ADMIN`
- **Required permission:** `SEE EXTRA CHARGE`
- **Input:** none
- **Output:** list of profile names (strings)

### `addExtraChargeProfile`
- **Auth type:** `ADMIN`
- **Required permission:** `CHANGE EXTRA CHARGE`
- **Input (dict):**
  - `profile_name` (str)
  - `comment` (str)
  - `effective_hour` (str, format `HH:MM`)
  - `rules` (list of dicts) – each rule contains:
    - `start_from` (choice: `user_creation_date`, `user_real_first_login`, `package_first_login`, `user_expiration_date`, `child_user_activation_fee`, `child_user_creation_date`, `child_active_user_creation_date`)
    - `description` (str)
    - `period` (choice: `monthly`, `start_of_month`, `daily`, `none`)
    - `period_value` (choice: `gregorian` or `jalali`)
    - `change_target` (choice: `credit`, `deposit`, `night_free`)
    - `change_action` (choice: `SET`, `ADD`, `MULTIPLY`)
    - `amount` (float)
    - `negate_credit` (bool)
    - `pay_back` (bool)
    - `ignore_first_time` (bool)
- **Output:** `int` – extra charge profile ID

### `updateExtraChargeProfile`
- **Permission:** `CHANGE EXTRA CHARGE`
- **Input:** same as above but includes `profile_id`, and each rule may have `rule_id` (use -1 for new rules).
- **Output:** `null`

### `deleteExtraChargeProfile`
- **Input:** `profile_name` (str)
- **Output:** `null`

### `getExtraChargeProfiles`
- **Permission:** `SEE EXTRA CHARGE`
- **Output:** list of profile dicts (including all rules)

### `getExtraChargeProfileByName` / `getExtraChargeProfileByID`
- **Input:** `profile_name` or `profile_id`
- **Output:** full profile dict

---

## Handler: `group`

User group management.

### `addNewGroup`
- **Auth type:** `ADMIN`
- **Required permission:** `CHANGE GROUP`
- **Input:** `group_name` (str), `comment` (str), `isp_name` (str)
- **Output:** `int` – Group ID

### `copyGroup`
- **Input:** `group_name`, `comment`, `isp_name`, `copy_count`
- **Output:** list of new Group IDs

### `listGroups`
- **Input:** optional `active_only` (bool), `isp_id` (comma‑separated)
- **Output:** list of group names

### `listGroupsWithIDs` (tag 320)
- **Input:** optional `active_only`
- **Output:** list of `[group_id, group_name]`

### `listGroupInfos` (tag 320)
- **Input:** optional `active_only`, `isp_id`
- **Output:** list of group info dicts (`group_id`, `group_name`, `comment`, `isp_id`, `isp_name`, `raw_attrs`, `attrs`)

### `getGroupCredits`
- **Output:** dict mapping group names to total credit

### `getGroupUsersCount`
- **Output:** dict mapping group names to user count

### `getGroupInfo`
- **Input:** either `group_name` or `group_id`
- **Output:** full group info dict (including attributes)

### `updateGroup`
- **Permission:** `CHANGE GROUP`
- **Input:** `group_id`, `group_name`, `comment`
- **Output:** `null`

### `updateGroupAttrs`
- **Permission:** `CHANGE GROUP`
- **Input:** `group_name`, `attrs` (dict), `to_del_attrs` (list)
- **Output:** `null`

### `delGroup`
- **Permission:** `CHANGE GROUP`
- **Input:** `group_name`
- **Output:** `null`

---

## Handler: `ippool`

IP pool management (including load balancing pools).

### `addNewIPpool`
- **Auth type:** `ADMIN`, **Permission:** `CHANGE IPPOOL`
- **Input:** `ippool_name` (str), `comment` (str)
- **Output:** `int` – IPPool ID

### `updateIPpool`
- **Input:** `ippool_id`, `ippool_name`, `comment`
- **Output:** `null`

### `getIPpoolNames`
- **Two variants:** without filter, or with `ippool_type` (empty string or `"load_balancing"`)
- **Output:** list of ippool names

### `getIPpoolInfo`
- **Permission:** `LIST IPPOOL`
- **Input:** `ippool_name`
- **Output:** dict with `ippool_id`, `ippool_name`, `comment`, `ip_list`, `free`, `used`

### `deleteIPpool`
- **Permission:** `CHANGE IPPOOL`
- **Input:** `ippool_name`
- **Output:** `null`

### `delIPfromPool`, `forceDelIPfromPool`, `addIPtoPool`
- Manage individual IPs in a pool.

### `addNewLoadBalancingIPpool`
- **Permission:** `CHANGE IPPOOL`
- **Input:** `ippool_name`, `ippool_comment`, `balancing_strategy` (`distributive` or `fill_first`), `children_ippool_percentages` (dict mapping pool name → percentage)
- **Output:** `null`

### `updateLoadBalancingIPpool`
- **Input:** `ippool_id`, `ippool_name`, `ippool_comment`, `balancing_strategy`, `children_ippool_percentages`
- **Output:** `null`

### `deleteLoadBalancingIPpool`
- **Input:** `ippool_name`
- **Output:** `null`

---

## Handler: `log_console`

Real‑time console logs for authentication/accounting events.

### `getConsoleBuffer`
- **Auth type:** `ADMIN`
- **Required permission:** `SEE CONNECTION LOGS`
- **Input:** none
- **Output:** list of lists, each containing: epoch time, formatted time, username, ISP name, action (e.g., `INTERNET_AUTHENTICATE`), RAS description, message.

---

## Handler: `login`

Authentication and session management.

### `webLogin` (tag 326)
- **Auth type:** `ANONYMOUS`
- **Input:** `login_auth_type` (`ADMIN`/`NORMAL_USER`/`VOIP_USER`), `login_auth_name`, `login_auth_pass`, `auth_remoteaddr`
- **Output (dict):** `session_id`, `auth_id` (username), `auth_type`

### `login` – with session creation
- **Auth type:** `ANONYMOUS`
- **Input:** `login_auth_type`, `login_auth_name`, `login_auth_pass`, `create_session` (true), `auth_remoteaddr`
- **Output:** `str` – Session ID

### `login` – without session (just authentication)
- **Input:** same without `create_session`
- **Output:** `bool` (true on success)

### `searchAdminLoginHistory`
- **Auth type:** `ADMIN`
- **Input (complex):**
  - `conds` dict with `admin`, `login_date_from`, `login_date_from_unit`, `login_date_to`, `login_date_to_unit`
  - `_from`, `to` (pagination)
  - `sort_by` (`admin_login_history_id` or `login_date`), `desc` (bool)
- **Output (dict):** `total_rows` and `report` list of login history entries.

---

## Handler: `mc` (Message Center)

SMS, email, and internal messaging.

### `getAllMSGTypes`, `getActiveMSGTypes`
- **Auth type:** `ADMIN, NORMAL_USER, VOIP_USER`
- **Input:** optional `view_type` (`SEND` or `REPORT`) for active types
- **Output:** list of message type strings

### Provider management (SMS/email providers)
- `getProviderTypes`, `getProviderInfoByName`, `getProviderAttrsByName`
- `addNewProvider` – requires `CHANGE PROVIDERS` permission
- `updateProviderAttrs`, `getAllActiveProviders`, `removeProvider`, `resetProviderAttrs`
- `addNewEPGroupToProvider`, `getAllEPGroupsOfProvider`, `getProviderUnassignedEPs`, `unAssingAllFromGroup`, `removeEPGroupFromProvider`, `getEPGroupUsageInfo`

### `receive` – receive a message (inbound)
- **Auth type:** `ADMIN, NORMAL_USER, VOIP_USER`
- **Input:** `msg_info` dict with `dst` (list), `msg_type`, `from`, `body`
- **Output:** `null`

### `enqueueSend` (user) and `adminEnqueueSend` (admin)
- **Two variants:** without schedule date, or with `schedule_date` and `schedule_date_unit` (jalali/gregorian/years/months/days/hours/minutes)
- **Input:** `body`, `destinations` (comma‑separated), `msg_type`
- **Output:** `null`

### Activation methods
- `activateService`, `resetUserActivationCode`, `checkActivationCode`, `getUserActivationNumber` – for user mobile verification.

### Search methods (received/sent/queued messages)
- **User‑side:** `userSearchReceivedMessage`, `userSearchSentMessage`, `userSearchQueuedMessage`, `userSearchMessagesForDialer`, `getUserLastMessageID`
- **Admin‑side:** `adminSearchReceivedMessage` (for users or ISPs), `adminSearchSentMessage`, `adminSearchQueuedMessage`

### Delete methods
- `dialerDeleteMessage`, `userDeleteMessage`, `adminDeleteMessage` – with `target_box` parameter.
- `userCancelScheduledMessages`, `adminCancelScheduledMessages`

### Remote request attributes (admin only)
- `updateAdminRemoteRequestAttrs`, `updateUserRemoteRequestAttrs`, `getRemoteRequestAttrs`


## Handler: `notification`

Notification profiles and rules for sending alerts (SMS, email, etc.) based on thresholds (expiration, credit, renew, birthday, online payment, etc.).

### `getNotificationProfileNames`
- **Auth type:** `ADMIN`
- **Output:** list of profile names the admin has access to.

### `getNotificationProfiles`
- **Required permission:** `CHANGE NOTIFICATION`
- **Output:** list of profile info dicts, each containing:
  - `notification_profile_id`, `notification_profile_name`, `notification_profile_comment`
  - `notification_rules` (list of rules with `notification_rule_id`, `notification_threshold`, `message_type`, `notification_type`, `message_template`)
  - `isp_id`, `owner_isp_name`

### `addNotificationProfile`
- **Permission:** `CHANGE NOTIFICATION`
- **Input:** `notification_profile_name`, `notification_profile_comment`
- **Output:** `int` – new profile ID

### `updateNotificationProfile`
- **Input:** `notification_profile_id`, `notification_profile_name`, `notification_profile_comment`
- **Output:** `null`

### `getNotificationProfileByName`
- **Input:** `notification_profile_name`
- **Output:** full profile dict (same structure as above)

### `deleteNotificationProfile`
- **Input:** `notification_profile_name`
- **Output:** `null`

### Rule management
- `addNotificationRule` – requires profile ID, notification type, threshold, message type, template
- `deleteNotificationRule` – requires profile ID and rule ID
- `updateNotificationRule` – requires rule ID, profile ID, and all rule fields

**Notification types:** `expiration_date`, `credit`, `renew`, `birth_day`, `online_payment`, `first_login`, `credit_change`, `deposit_change`

**Message types:** `sms`, `email`, `email_to_isp`, `ibsng_message`, `url`, `user_event`

---

## Handler: `online_payment`

Payment gateway integration and transaction handling.

### User/admin accessible methods

#### `getAvailableGatewayTypes`
- **Auth type:** `ADMIN, NORMAL_USER, VOIP_USER`
- **Output:** list of gateway type strings (e.g., `Eghtesad_Novin`, `Melli-shahparak`, `Mellat-Shaparak`, `Parsian`, `Pasargad`, `Saman`, `Tejarat`, `ZarinPal`)

#### `getPaymentAmountSuggestions`
- **Output:** list of suggestions, each a list of 4 items: suggestion_id (`renew_credit`, `recharge_credit`, `arbitrary`, `recharge_from_renew_deposit`), amount (float), description (str), editable (bool)

#### `preparePayment` (tag 326)
- **Input:** `gateway_type`, `amount`, `unique_id`, `callback_url`, `attributes` (dict; can include `no_action` to skip automatic credit/deposit after verification)
- **Output:** `web_attributes` (dict), `redirect_method` (`POST`/`GET`), `url` (str)

#### `verifyPayment` (tag 326)
- **Input:** `unique_id`, `web_attributes`
- **Output:** `amount` (float), `ref_id` (bank reference ID)

### Admin-only gateway management

#### `addGateway`
- **Permission:** `CHANGE ONLINE PAYMENT`
- **Input:** `gateway_name`, `gateway_type`, `owner_isp_name`, `priority`, `comment`, `attributes` (dict)
- **Output:** `null`

#### `updateGateway`, `deleteGateway`
- Similar inputs, using `gateway_id` or `gateway_name`.

#### `getAllGatewayTypes`
- **Permission:** `SEE ONLINE PAYMENT`
- **Output:** list of gateway type strings

#### `getAllGatewayNames`
- **Output:** dict with `gateway_names` list

#### `getAllGatewayInfos`
- **Permission:** `CHANGE ONLINE PAYMENT`
- **Output:** list of gateway info dicts (including all attributes)

#### `purgePayment`
- **Permission:** `PURGE ONLINE PAYMENT`
- **Input:** `payment_id`
- **Output:** dict (details of purged payment)

---

## Handler: `ras`

RAS (Remote Access Server) / NAS (Network Access Server) management.

### `addNewRas`
- **Auth type:** `ADMIN`, **Permission:** `CHANGE RAS`
- **Input:** `ras_ip`, `ras_type` (ACS2, Cisco-IN, CiscoVPDN, Generic, Huawei, Mikrotik, Ruckus, ZTE), `radius_secret`, `ras_description`, `comment`
- **Output:** `int` – Ras ID

### `getRasInfo`
- **Permission:** `GET RAS INFORMATION`
- **Input:** `ras_ip`
- **Output:** dict with all RAS attributes including `attrs`

### `getActiveRasIPs`
- Two variants: without filter (all active), or with `type_filter` (comma‑separated)
- **Output:** list of RAS IPs

### `getRasDescriptions`
- **Output:** list of `[ras_description, ras_ip]`

### `getInActiveRases`
- **Permission:** `LIST RAS`
- **Output:** list of inactive RAS IPs

### `getRasTypes`
- **Output:** list of RAS type strings

### `getRasAttributes`
- **Input:** `ras_ip`
- **Output:** dict of RAS attributes

### `updateRasInfo`
- **Input:** `ras_id`, `ras_ip`, `ras_type`, `radius_secret`, `ras_description`, `comment`
- **Output:** `null`

### `updateAttributes`, `resetAttributes`
- Manage RAS-specific attributes.

### `deActiveRas`, `reActiveRas`, `deleteRas`
- Change RAS status.

### IP pool assignment to RAS
- `getRasIPpools` – returns sorted list of IPPool names for a RAS
- `addIPpoolToRas`, `delIPpoolFromRas`

---

## Handler: `report`

Comprehensive reporting – online users, connection logs, credit changes, usage summaries, etc.

### Online users

#### `getOnlineUsers`
- **Permission:** `SEE ONLINE USERS`
- **Input (complex):**
  - `normal_sort_by` / `voip_sort_by` (various fields)
  - `normal_desc` / `voip_desc` (bool)
  - `conds` dict with optional filters: `dnis`, `isp_ids`, `isp_names`, `ras_ips`, `remote_ip`, `username_starts_with`
- **Output:** list of two items:
  - Index 0: list of internet online user dicts (user_id, service, ras_ip, login_time, duration_secs, attrs, current_credit, group_name, etc.)
  - Index 1: list of VoIP online user dicts

#### `getOnlineUsersCountLoop` (with conditions) and `getOnlineUsersCount` (without conditions)
- **Output:** dict with counts: `internet_onlines`, `voip_onlines`, `re_onlined`, `accounting_not_started`, `failed_users`, `idle_users`, plus breakdown by RAS, group, ISP.

#### `getOnlineUsersCountPerISP`
- **Output:** `onlines_per_isp` dict mapping ISP ID to count.

### Connection logs

#### `getConnections` (admin and user versions)
- **Permission (admin):** `SEE CONNECTION LOGS`
- **Extensive input** (conds with many filters: user_ids, username, credit_used, duration, login_time range, logout_time range, successful, service, ras_ip, caller_id, mac, port, assigned_ip, station_ip, called_number, prefix_name, voip_provider_names, etc.)
- **Pagination:** `from`, `to` (indices)
- **Sort by:** various fields
- **Output (dict):** `total_rows`, `total_credit`, `total_duration`, `total_in_bytes`, `total_out_bytes`, `report` list of connection dicts.

#### `getINConnections` (admin only) – requires `caller_id` in conds.

### Usage aggregations

#### `getDurations`
- **Output:** list of 6 ranges with counts of connections in each duration bucket.

#### `getGroupUsages`, `getRasUsages`, `getISPUsages`
- **Output:** list of `[name, total_duration_seconds]`

#### `getVoIPDisconnectCauses`
- **Output:** list of `[cause_code, count]`

#### `getSuccessfulCounts`
- **Output:** `[successful_count, failed_count]`

#### `getInOutUsages`, `getCreditUsages`, `getDurationUsages`, `getConnectionUsages`
- Per-user aggregated reports (download/upload, credit, time, combined).

### Expired users

#### `getExpiredUsers`, `searchExpiredUsers`, `searchExpiredUsersExtended`
- Various filters (ISP, group, expiration date range)
- **Output:** total count, credit sum, and list of user IDs (or dict of user_id → expiration date).

### Credit changes

#### `getCreditChanges` – three versions: admin-view, admin user-view, and user self-view.
- **Input:** conds with user_ids, admin, action list, credit amount filters, date range, etc.
- **Output:** `total_rows`, `report` list of credit change records, plus `total_per_user_credit` and `total_admin_credit` (or `total_isp_credit`).

#### `saveCreditChanges` – export to CSV/PDF.

### Deposit changes

#### `getUserDepositChanges` (admin and user versions)
- Similar filters, returns deposit change logs.

### Audit logs

#### `getUserAuditLogs` – attribute change history for users/groups.
#### `getSystemAuditLogs` – system-wide audit (categories: RAS, CHARGE, ISP, VOIP-TARIFF, ADMIN, etc.).

### Online payment report

#### `getOnlinePaymentReport`
- **Auth type:** `ADMIN, NORMAL_USER, VOIP_USER`
- **Input:** conds with requester_type, amount, payment date range, succeed flag, gateway_name, etc.
- **Output:** paginated report of payments.

### Management summary

#### `getManagementSummaryReport`
- **Permission:** `SEE CONNECTION LOGS`
- **Input:** requires `login_time_from`, `view_period` (daily/weekly/monthly/yearly), `included_objects` (isp, group, ras, etc.), `report_targets` (duration, credit, in_bytes, out_bytes)
- **Output:** nested structure with date ranges and aggregated values.

### Report saving (export to CSV/PDF)

Methods that spawn background report generation; use `SystemNotification.getNotifications()` to check completion and get download URL:
- `saveCreditChanges`
- `saveConnections`
- `saveConnectionUsages`
- `savePrefixNameUsage`
- `saveOnlinePaymentReport`
- `saveSearchExpiredUsers`

### Other report utilities

- `delReports`, `autoCleanReports`, `getAutoCleanDates` – manage report data retention.
- `getRequestCount`, `getRequestTopStats` – admin request statistics (requires `GOD` permission).
- `getISPDepositChangeLogs`, `getTemporaryExtendLogs` – specific logs.
- `getDayNightUsage` – nightly vs daytime traffic usage (requires `GOD`).


## Handler: `session`

Session management.

### `expireSession`
- **Auth type:** `ADMIN, NORMAL_USER, VOIP_USER`
- **Input:** `session_id` (str)
- **Output:** `null`

### `getAuthData`
- **Input:** `auth_session` (session ID)
- **Output:** list of 3 items: `auth_name`, `auth_pass`, `auth_type`

---

## Handler: `snapshot`

Real-time and historical bandwidth/online user snapshots.

### `getBWSnapShotForUserAjax`
- **Admin version:** requires `SEE REALTIME SNAPSHOTS`, input `user_id`, optional `from` and `from_unit`
- **User version:** no user_id required
- **Output:** list of `[epoch_time, download_rate, upload_rate]`

### `getOnlinesSnapShot` (admin only)
- **Permission:** `SEE ONLINE SNAPSHOTS`
- **Input:** `conds` (ras_ips, isp_names), `type` (`internet` or `voip`)
- **Output:** list: result (list of `[epoch, count]`), minimum, maximum, average, timezone string.

### `getBWSnapShot`
- **Auth type:** `ADMIN, NORMAL_USER, VOIP_USER`
- **Input:** `conds` dict with `user_id`, `date_from`, `date_to`, date unit and operator options
- **Output:** same as above (epoch, download, upload)

### Group/ISP/RAS bandwidth snapshots
- `getGroupBWSnapShot`, `getISPBWSnapShot`, `getRasBWSnapShot` – admin only
- Each takes `conds` with appropriate name lists and date range parameters.

---

## Handler: `stat`

System statistics.

### `getStatistics`
- **Auth type:** `ADMIN`, **Permission:** `GOD`
- **Output:** list of `[stat_key, [value, unit]]` (23 items)

### `getAllStatsByStatCategory`
- **Output:** dict mapping category → stats with 5 time windows (5min, 1hour, 8hour, 1day, 1week), each with value and count.

### `getUserQueueStats`
- **Output:** list of `[ras_description or "TOTAL", dict of queue counters: auth, auth_reonline, can_stay, other, stop, update]`

---

## Handler: `SystemNotification`

In-system notifications for background tasks (e.g., report generation completion).

### `getNotifications`
- **Auth type:** `ADMIN`
- **Input:** optional `last_notifications` (default 5), `only_unread` (bool, default false)
- **Output:** list of notification dicts with `notification_id`, `message`, `date`, `read` (t/f), `admin_id`, `links`, `type`

### `changeNotificationStatus`
- **Input:** `notification_id`, `read` (bool)
- **Output:** number of updated notifications

---

## Handler: `telephony_support`

Telephony integration for support systems (call centers).

### `callerIDAuthenticate`
- **Permission:** `TELEPHONY SUPPORT`
- **Input:** `caller_id`
- **Output:** `[user_id, credit, language]` (language: `fa` or empty)

### `authenticate`
- **Input:** `auth_by` (`user_id`, `serial`, `voip_username`, `internet_username`, `phone`), `auth_id`
- **Output:** `[user_id, credit]`

### `getCurrentCredit`
- **Input:** `user_id`
- **Output:** float (credit)

### `getRemainingByteDuration`
- **Input:** `user_id`
- **Output:** `[status (0=Package, 1=Recharged, 2=Temporary Extended), dict of charge_rule_desc → [duration, bytes, state, optional extra info]]`

### `getNearestExpDate`
- **Input:** `user_id`
- **Output:** expiration date string

### `getFailureReason`
- **Input:** `user_id`
- **Output:** integer code (0=success, 900‑920 for various failures)

### `getLastConnection`
- **Input:** `user_id`
- **Output:** list `[login_time, duration_seconds, credit_used, successful]` (empty if none)

### `recharge`
- **Input:** `user_id`, `pin`
- **Output:** float – increased credit

### `checkInternetPassword`, `changeInternetPassword`
- Verify or change internet password.

---

## Handler: `user`

User management – the largest handler.

### Adding users

#### `addNewUsers`
- **Permission:** `ADD NEW USER`
- **Input:** `count` (int), `credit` (dict of credit_index → value), `isp_name`, `group_name`, `credit_comment`, optional `custom_fields`
- **Output:** list of new user IDs

### Getting user info

#### `getUserInfo` – User self‑view
- **Auth type:** `NORMAL_USER, VOIP_USER`
- **Output:** extensive dict with `basic_info` (credit, deposit, group, ISP, nearest_exp_date, status) and `attrs` (all user attributes including custom fields, internet/voip credentials, expiration dates, flags, billing settings, etc.)

#### `getUserInfo` – Admin view
- **Permission:** `GET USER INFORMATION`
- **Input:** one of `user_id`, `normal_username`, `voip_username`, `serial`, `phone`, `remote_ip` (each can be multi‑value)
- **Output:** dict mapping user ID → same user info dict as user self‑view

### User existence

#### `doesUserExists`
- **Input:** `normal_username`
- **Output:** bool

### Updating users

#### `updateUserAttrs`
- **Permission:** `CHANGE USER ATTRIBUTES`
- **Input:** `user_id` (string, multi‑value), `attrs` (dict of attributes to update – extremely flexible, includes internet/voip credentials, custom fields, flags, expiration dates, bandwidth, notification profiles, etc.), `to_del_attrs` (list)
- **Output:** `null`

### Credit and deposit changes

#### `changeCredit`
- **Permission:** `CHANGE USER CREDIT`
- **Input:** `user_id` (multi), `credit` (float, credit 1), `is_absolute_change` (bool), `credit_comment`
- **Output:** list of change log IDs

#### `changeCreditExtended`
- **Input:** `user_id` (multi), `credit` (dict of credit_index→value), `change_type` (`ADD`/`SET`/`MULTIPLY`), `credit_comment`
- **Output:** list of change log IDs

#### `changeDeposit`, `changeDepositExtended`
- Same pattern for deposit.

### Searching users

#### `searchUser`
- **Permission:** `SEARCH USER`
- **Extensive `conds`** (over 100 possible filters: absolute/relative expiration dates, username patterns, group, ISP, credit amount, deposit, lock status, online status, custom fields, charge name, notification profile, multi‑login, MAC, caller ID, etc.)
- **Pagination:** `from`, `to`
- **Sort by:** various fields
- **Output:** `[total_rows, total_credit_sum, list_of_user_ids]`

#### `saveSearchUser` – export to CSV/PDF (background task)

#### `searchExpiredUsers`, `searchExpiredUsersExtended`, `saveSearchExpiredUsers`
- Find users with expiration conditions.

### Deleting and killing users

#### `delUser`
- **Permission:** `DELETE USER`
- **Input:** `user_id` (multi), `delete_comment`, `del_connection_logs` (bool), `del_audit_logs` (bool)
- **Output:** `null`

#### `killUser` (admin) – requires `KILL USER` or `CLEAR USER`
- **Input:** `user_id` (multi), `ras_ip` (multi), `unique_id_val`, `kill` (bool, default true)
- **Output:** `null`

#### `killUserByID` – admin version kills all instances; user version kills own all instances.

#### `killMe` – user disconnects a specific instance (requires `ras_ip`, `unique_id_val`).

### Renew, recharge, temporary extend

#### `renewUsers` – renews user packages
#### `rechargeUsers` – adds recharge credit
#### `temporaryExtendUsers` – extends expiration by hours and adds credit

### Bulk actions

#### `bulkUpdateUserAttrs`, `bulkChangeUserCredit`, `bulkChangeUserDeposit`, `bulkDeleteUser`, `bulkRenewUsers`, `bulkKillUsers`
- Combine `searchUser` with respective action.
- All return `action_id` (string) for tracking.

#### `getBulkActionsForAdmin`, `viewBulkActionStatus`, `cancelBulkAction`, `clearBulkAction`

### Other user methods

- `getUsersWithPhone`, `getUsersWithCellPhone`
- `getRemainingDurationAndBytes` (admin and user)
- `changeStatus` (Package/Recharged/Temporary Extended)
- `updateUserComments` (user self‑edit)
- `reloadUsers` (force reload from database)
- `setOneChargeRuleUsage`, `setFeshfesheParams`
- `getUsersExpDateFirstLogin` (large output, for CRM)

---

## Handler: `user_custom_field`

Define and manage custom fields for user profiles.

### `addNewCustomField`
- **Permission:** `CHANGE USER CUSTOM FIELDS`
- **Input:** `name`, `description`, `comment`, `value_type` (string/int/float), `interface_type` (text_field/single_select/radio_button/checkbox), `allowable_values` (list), `mandatory` (bool)
- **Output:** `null`

### `updateCustomField`, `deleteCustomField`, `getAllCustomFields`
- Similar inputs.

---

## Handler: `ldap`

LDAP integration (read‑only user import).

### `getUsernames`, `getUserInfos`
- **Input:** `starts_with`
- **Output:** list of usernames or user info dicts.

### `getUserInfo`
- **Input:** `username`
- **Output:** user info dict

### `getUserInfoKeys`
- **Input:** `domain`
- **Output:** list of domain info keys

### Mapping
- `setLDAPIBSMapping`, `getLDAPIBSMapping`, `deleteLDAPIBSMapping` – map LDAP attributes to IBSng fields.
- `setLDAPSearchOptionsByDomain`, `getLDAPSearchOptionsByDomain` – search filter and base.

---

## Handler: `util`

Utility methods.

### `multiStrGetAll`
- Splits multi‑string parameters.

### `runDebugCode` (requires `GOD`)

### `getStartOfMonth`, `getNow`

### AFE remote admin methods
- `afeGetUserInfo`, `afeGetAllGroups`, `afeChangeUserCredit`

### IP‑based session and user info
- `getUserIDForIP`, `getUsernameForIP` (requires `SEE ONLINE USERS`)
- `createSessionForFailedUserByIP`, `createSessionForUserByIP`
- `getErrorForFailedUser`, `kickFailedUserByIP`, `kickIDLEUsersByIP`

### Page style helpers
- `getSessionISPID`, `getISPsPageStyleRevision`, `getSessionPageStyle`

### System info
- `getDRBDStatus` (requires `GOD`)
- `getSavedReportList`, `deleteSavedReport`
- `version` – returns `current_tag`, `version`, `branch`
- `echo` – returns same string (testing)

---

## Handler: `voip_provider`

VoIP provider and routing profiles (minimal documented methods).

### `listRoutingProfiles`
- **Permission:** `SEE VOIP PROVIDER`
- **Output:** list of profile names

### `listVoIPProviders`
- **Output:** list of provider names

---

## Handler: `voucher`

Voucher management (PIN‑based recharge or user creation).

### Searching

#### `searchVoucher`
- **Permission:** `SEE VOUCHER`
- **Input:** conds (voucher_ids, batch_ids, pins, is_used, isp_names), pagination, sort_by (`pin`/`voucher_id`), `desc`
- **Output:** `total_rows`, `report` list of voucher details.

#### `searchBatch` (admin)
- Similar conds (isp_names, batch_ids, batch_names, is_locked, can_recharge_user, can_create_user, credit range)
- Output batch info.

### Using vouchers

#### `voucherRechargeUser`
- **Admin:** requires `USE VOUCHER ON USERS` + `user_id`
- **User:** requires `allow_recharge_by_voucher` flag
- **Input:** `voucher_pin` (and `user_id` for admin)
- **Output:** float – voucher credit

#### `voucherAddNewUser` (admin)
- **Permissions:** `USE VOUCHER ON USERS, ADD NEW USER`
- **Input:** `voucher_pin`, `isp_name`, `group_name`
- **Output:** new user ID

#### `getBatchVoucherAttrs`
- Returns full details of voucher and its batch.

### Batch management

#### `voucherAddBatch`
- **Permission:** `CHANGE VOUCHER`
- **Input:** `batch_dict` (name, is_locked, credit, can_recharge_user, can_create_user, change_target, allow_use_by_children_isp, comment), `pin_prefix`, `pin_len`, `serial_prefix`, `serial_start`, `count`
- **Output:** batch_id

#### `voucherChangeBatchLockStatus`, `voucherGetBatchByID`, `getVoucherByID`, `voucherSearchBatch`

---

## Handler: `invoice` (based on branch `C_invoice`)

Invoice and proforma invoice management.

### Invoice profiles

#### `getInvoiceProfileNames`, `getInvoiceProfiles`, `getInvoiceProfileByName`, `getInvoiceProfileByID`, `getInvoiceProfileByUserID`, `getInvoiceProfileByGroupName`
- **Permission:** `SEE INVOICE PROFILE`
- Output full profile definitions (rules, rule items, templates, actions, etc.)

#### `getInvoiceRuleByID`

#### `addInvoiceProfile` (permission `CHANGE INVOICE PROFILE`)
- Input: `profile_name`, optional `isp_name`, `comment`
- Output: profile ID

#### `updateInvoiceProfile`, `deleteInvoiceProfile`

### Templates
- `getAllTemplateNames`, `addInvoiceTemplate`, `deleteInvoiceTemplate`

### Searching invoices and proformas

#### `searchInvoices` (admin and user)
- **Admin permission:** `SEE INVOICE`
- Input conds: `isp_names`, `user_ids`, `invoice_ids`, `is_active`, `is_paid`, `total_amount_from/to`, `issue_date_from/to`
- Output: paginated report.

#### `searchProformaInvoices` (admin)
- Similar with `pi_ids`, `is_expired`.

### Issuing and paying

#### `issueProformaInvoice`
- **Input:** `attrs_list` (list of dicts with `user_id`, `rule_id`, `comment`, `add_user`, `add_user_details`), `arbitrary_amount`
- Output: dict with `issued_pis` list and `failed_users`

#### `invoicePutUserOnDept`, `proformaInvoicePaid`, `invoicePaid`
- Mark invoices/proformas as paid, apply actions.

#### `getPIByID`, `getInvoiceByID`, `getPIWithRuleByPIID`, `getInvoiceWithRuleByInvoiceID`
- Retrieve detailed invoice/proforma with associated rule.
