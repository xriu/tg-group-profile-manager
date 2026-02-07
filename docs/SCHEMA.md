## Profile configuration schema

The schema consists of an object with the following properties:

- _profiles_: List of Object, where each Object defines a Profile
- _groupPermissions_: Dictionary, where each key is a group name and value is a requisite group name for being able to select it. _Optional_.
- _notification_: Object, controls notification behavior. _Optional_.
  - _expired_: Object, controls expiration notification behavior. _Optional_.
    - _message_: Boolean, when `false` no expiration message is sent. _Default_: `true`
    - _mention_: Boolean, when `false` the expiration message will not include `@mentions` (no ping). Only applies when `message` is `true`. _Default_: `true`

### Profile:

This object defines a single profile

- _profileName_: String, user-friendly profile name. _Required_.
- _profileType_: Enum, only `oneOf` and `selfServeApproval` supported currently. _Default_: `oneOf`
- _applicableToGroup_: String, a Twingate group that a user must be in for them to see this profile. _Default_: "Everyone"
- _approverGroup_: String, `selfServerApproval` specific, a Twingate group a user must be in for them to approve the access request of the Profile. The group size should not exceed 20 members. _Required_.
- _timeOptions_: Enum, `selfServerApproval` specific, the duration options requesters can select during access request. Must be one of `Forever`, `1h`, `2h`, `4h`, `6h`, `8h`, `12h`, `24h`, `2d`, `3d`,`4d`, `5d`, `6d`, `7d`, `14d`,`21d`,`30d`, `60d`, `90d`. _Default_: `[Forever]`
- _notificationGroups_: Array of String, `selfServeApproval` specific, Twingate group names whose members receive a read-only notification DM when access is granted (self-approved or approved). No approve/reject buttons. _Optional_.
- _groups_:
  - _oneOf_: List of String, Twingate groups within the profile which the users can switch between. _Required_.
  - _selfServeApproval_: List of String, Twingate groups within the profile which the users can request access to. _Required_

### Example:

```json
{
  "profiles": [
    {
      "profileName": "Example Profile 1",
      "profileType": "oneOf",
      "groups": ["Prod", "Preprod", "Testing"],
      "applicableToGroup": "Everyone"
    },
    {
      "profileName": "Example Self-Serve Business Approvals",
      "profileType": "selfServeApproval",
      "groups": ["HR", "Finance", "Sales"],
      "timeOptions": ["Forever", "1h", "8h", "24h", "7d", "30d", "90d"],
      "applicableToGroup": "Everyone",
      "approverGroup": "IT",
      "notificationGroups": ["SRE"]
    }
  ],
  "groupPermissions": {
    "Prod": "Admin"
  },
  "notification": {
    "expired": {
      "message": true,
      "mention": true
    }
  }
}
```

In the example above:

**Example Profile 1**

- Any user in the `Everyone` group can access the `Example Profile 1` Profile
- Any user that can access the profile can choose from `Preprod` or `Testing` groups via the Slackbot
- In order to select the `Prod` group a user must _also_ be in the requisite group named `Admin` because of the `groupPermission` object.

**Example Self-Serve Business Approvals**

- Any user in the `Everyone` group can access the `Example Self-Serve Business Approvals` Profile
- Any users that can access the profile can request access to groups `HR`, `Finance` and `Sales`
- Any users that can access the profile can select access duration `Forever`, `1h`, `8h`, `24h`, `7d`, `30d` and `90d`
- The request can be only be approved by the group `IT` members
- Members of the `SRE` group will receive a read-only notification DM when access is granted (no approve/reject buttons)