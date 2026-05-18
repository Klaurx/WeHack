# Admin Panel Surface Map

Reconstructed from the compiled class listing in the 试玩测试 build output and the JSP page directory structure in both CPL and 试玩测试 workspaces. This represents the full controller and view surface of the `fastshiwan-web-backstage` application. The CPL backstage (`fastcpl-web-backstage`) has a similar but smaller surface.

---

## Authentication and Session

The application uses Apache Shiro for session management with a custom `ShiroDbRealm` implementation. See F-06 for filter chain misconfiguration details.

A separate `ApiLoginFilter` handles authentication for the mobile API layer (`/api/**`). This is a custom implementation not governed by Shiro and requires independent audit.

The `WaimaiApiLoginFilter` handles authentication for the food delivery subsystem (`/waimai/api/**`), also custom.

Session timeout is configured at 60 minutes in `web.xml`.

---

## Admin Controllers (require authentication via `/**=authc`)

### System Management
- `SystemUserController` - admin user management
- `RoleController` - role management
- `PermissionController` - permission management
- `SystemLogController` - operation log viewer
- `AdminOperationLogService` - operation audit trail

### User Management
- `UserController` - user listing and management
- `UserExtController` - extended user data
- `UserAlipayIdCardController` - user Alipay identity verification records
- `UserChannelGameController` - user game channel assignments
- `UserFengKongReportController` - risk/fraud report management
- `UserShuaziLikeGameController` - bot-detection game preferences

### Financial / Payment
- `OrderWithdrawController` - withdrawal order management
- `ApiUserPayController` - payment initiation
- Multiple Alipay integration controllers (see Payment Integrations document)

### Channel Management
- `ChannelGameManagementController` - game channel management
- `ChannelGameV2Controller` - v2 channel management
- `ChannelGameTagController` - channel game tagging
- `ChannelGameCountRankController` - channel performance ranking
- `CplchannelController` - CPL channel management
- `CplchannelMediaController` - CPL media management

### Platform / Game Management
- `MoguController` - Mogu integration management
- `CalendarController` - calendar/scheduling
- `ClientConfController` - client configuration
- `FileUploadController` - file upload handling
- `CommonController` - shared utilities
- `AutoMappingController` - automatic route mapping
- `WechatConf` - WeChat configuration management
- `OaidCert` - OAID certification management

### Activity and Promotions
- `ActivityShuaziTixanController` - bot detection withdrawal activity
- `PromoteActivityController` - promotion activity management
- `PromoteActivityCountController` - promotion analytics
- `ApiActivityFangDanController` - anti-cheat activity
- `ApiActivityShuaZiController` - bot activity management
- `ApiActivityWeekRankController` - weekly ranking activity
- `ApiActivityMonthSettlementController` - monthly settlement activity
- `ApiActivityXiaomanController` - Xiaoman activity
- `ApiActivityYuyinhongbaoController` - voice red packet activity
- `ActivityTuziDayRechargeFuliController` - daily recharge welfare

### CPS (Cost Per Sale) Subsystem
- `CPSMobileController` - CPS mobile interface
- `QingtingAPI` + `QingtingTask` - QingTing integration
- `ApiCpsDayRechargeFuliController` - CPS daily recharge welfare

### Food Delivery / Waimai Subsystem
- `WaimaiApiUserController` - waimai user API
- `WaimaiApiUserAlipayAccountController` - waimai Alipay accounts
- `ElemeTask` - Ele.me task handler
- `MeituanTask` - Meituan task handler
- `TbkOrder` / `TbkQudao` - Taobao Ke order handling

---

## Unauthenticated API Controllers (Shiro `anon`)

These controllers handle requests without Shiro authentication. Each has its own authentication mechanism (token, signature, or none).

### Mobile App API (`/api/**`)
- `ApiUserController` - core user operations, payment channel enum
- `ApiUserAppClientController` - app client registration
- `ApiUserCardController` - user card data
- `ApiUserChannelGameController` - channel game access
- `ApiUserClientController` - client management
- `ApiUserFKReportController` - risk reporting
- `ApiUserGameController` - game access
- `ApiUserIdCardController` - identity card operations
- `ApiUserSignInController` - sign-in/check-in
- `ApiUserVedioController` - video content
- `ApiHomeController` - home page data
- `ApiBlackGameController` - blacklisted game management
- `ApiUserFollowGameController` - game follow/favorite
- `ApiUserDayFuliController` - daily welfare
- `ApiUserDayRechargeFuliController` - daily recharge welfare
- `ApiUserTuiguangController` - referral/promotion
- `ApiUserShuaiGameDetailController` - game detail
- `ApiQudaoController` - channel data
- `ApiUserFengkongController` - risk control with Aliyun fingerprint integration

### Channel Notification Callbacks (`/channel/**`)
Each of the following receives webhook callbacks from the respective game channel. All require signature verification using the keys listed in the credential inventory.

- `ChannelABXNotifyController`
- `ChannelXYXNotifyController` + `ChannelNewXYXNotifyController`
- `ChannelDDZNotifyController`
- `ChannelDuoyouNotifyController`
- `ChannelJHLNotifyController`
- `ChannelJXWNotifyController`
- `ChannelMEITUANNotifyController`
- `ChannelMOGuXQNotifyController`
- `ChannelMiniGameXQNotifyController`
- `ChannelQWNotifyController`
- `ChannelTaoJinNotifyController`
- `ChannelXIANWNotifyController`
- `ChannelXQNotifyController`
- `ChannelXWNotifyController`
- `ChannelYouLeNotifyController`
- `ChannelYPJZNotifyController`
- `ChannelCPLNotifyController`
- `ChanglianService` + `PayController` (ChangLian payment)

### App Native API (`/m/**` and app controllers)
- `AppApiUserController`
- `AppApiUserTixianController` - withdrawal
- `AppABXNotifyController`
- `AppMOGuXQNotifyController`
- `AppMobileController`
- `AppApiUserDayFuliController`
- `MobileController`

---

## Background Tasks (Scheduled)

The following scheduled tasks run automatically and interact with external services using the credentials from prod.properties:

- `ABXTask` - ABX channel sync (9 inner classes, indicating complex multi-step process)
- `DDZTask` - DDZ channel sync (8 inner classes)
- `DuoyouTask` - DuoYou channel sync (9 inner classes)
- `JHLTask` - JHL channel sync
- `JXWTask` - JXW channel sync
- `NEWXYXTask` - New XYX channel sync
- `XQTask` - XQ channel sync
- `XWTask` - XW channel sync (6 inner classes)
- `XYXTask` - XYX channel sync
- `XianWTask` - XianWan channel sync
- `TaojinTask` - TaoJin channel sync
- `QWTask` - QW channel sync
- `YouleTask` - YouLe channel sync
- `CPLTask` - CPL self-channel sync
- `DuoyouCplChannelTask` - DuoYou CPL sync
- `ActivityYuYinHongBaoTask` - voice red packet activity
- `ArchiveUserCoinDetTask` - coin archive task
- `CheckUserBaseTask` - Tencent anti-fraud user check
- `AliyunDNS` - Aliyun DNS management task
- `WxTask` - WeChat task handler
- `QingtingTask` - QingTing integration task
- `UserSettleTask` - user settlement processing
- `UserExtTask` - user extension data sync
- `TutuGameTask` - TuTu game integration
- `QudaoTask` - channel task handler
- `ShuaziTask` - bot detection task
- `OnlineUserManger` - online user tracking

All of these tasks use credentials from the properties files documented in F-04.
