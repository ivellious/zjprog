# zjprog
Zajecia 1 - analiza kompleksowości kodu
język JAVA - narzędzie checkstyle -> liczenie Cyclomatic Complexity ->
The complexity is equal to the number of decision points + 1 Decision points: if, while , do, for, ?:, catch , switch, case statements, and operators && and || in the body of target.
By pure theory level 1-4 is considered easy to test, 5-7 OK, 8-10 consider re-factoring to ease testing, and 11+ re-factor now as testing will be painful. 

## metoda w package RxJava
[ERROR] /home/mpom/Downloads/../Documents/RxJava/src/main/java/io/reactivex/internal/operators/observable/ObservableFlatMap.java:330:9: Cyclomatic Complexity is 40 (max allowed is 10). [CyclomaticComplexity]
```
void drainLoop() {
            final Observer<? super U> child = this.downstream;
            int missed = 1;
            for (;;) {
                if (checkTerminate()) {
                    return;
                }
                SimplePlainQueue<U> svq = queue;

                if (svq != null) {
                    for (;;) {
                        if (checkTerminate()) {
                            return;
                        }

                        U o = svq.poll();

                        if (o == null) {
                            break;
                        }

                        child.onNext(o);
                    }
                }

                boolean d = done;
                svq = queue;
                InnerObserver<?, ?>[] inner = observers.get();
                int n = inner.length;

                int nSources = 0;
                if (maxConcurrency != Integer.MAX_VALUE) {
                    synchronized (this) {
                        nSources = sources.size();
                    }
                }

                if (d && (svq == null || svq.isEmpty()) && n == 0 && nSources == 0) {
                    Throwable ex = errors.terminate();
                    if (ex != ExceptionHelper.TERMINATED) {
                        if (ex == null) {
                            child.onComplete();
                        } else {
                            child.onError(ex);
                        }
                    }
                    return;
                }

                boolean innerCompleted = false;
                if (n != 0) {
                    long startId = lastId;
                    int index = lastIndex;

                    if (n <= index || inner[index].id != startId) {
                        if (n <= index) {
                            index = 0;
                        }
                        int j = index;
                        for (int i = 0; i < n; i++) {
                            if (inner[j].id == startId) {
                                break;
                            }
                            j++;
                            if (j == n) {
                                j = 0;
                            }
                        }
                        index = j;
                        lastIndex = j;
                        lastId = inner[j].id;
                    }

                    int j = index;
                    sourceLoop:
                    for (int i = 0; i < n; i++) {
                        if (checkTerminate()) {
                            return;
                        }

                        @SuppressWarnings("unchecked")
                        InnerObserver<T, U> is = (InnerObserver<T, U>)inner[j];
                        SimpleQueue<U> q = is.queue;
                        if (q != null) {
                            for (;;) {
                                U o;
                                try {
                                    o = q.poll();
                                } catch (Throwable ex) {
                                    Exceptions.throwIfFatal(ex);
                                    is.dispose();
                                    errors.addThrowable(ex);
                                    if (checkTerminate()) {
                                        return;
                                    }
                                    removeInner(is);
                                    innerCompleted = true;
                                    j++;
                                    if (j == n) {
                                        j = 0;
                                    }
                                    continue sourceLoop;
                                }
                                if (o == null) {
                                    break;
                                }

                                child.onNext(o);

                                if (checkTerminate()) {
                                    return;
                                }
                            }
                        }

                        boolean innerDone = is.done;
                        SimpleQueue<U> innerQueue = is.queue;
                        if (innerDone && (innerQueue == null || innerQueue.isEmpty())) {
                            removeInner(is);
                            if (checkTerminate()) {
                                return;
                            }
                            innerCompleted = true;
                        }

                        j++;
                        if (j == n) {
                            j = 0;
                        }
                    }
                    lastIndex = j;
                    lastId = inner[j].id;
                }

                if (innerCompleted) {
                    if (maxConcurrency != Integer.MAX_VALUE) {
                        ObservableSource<? extends U> p;
                        synchronized (this) {
                            p = sources.poll();
                            if (p == null) {
                                wip--;
                                continue;
                            }
                        }
                        subscribeInner(p);
                    }
                    continue;
                }
                missed = addAndGet(-missed);
                if (missed == 0) {
                    break;
                }
            }
}
```

## metoda w package AndroidDPC-test 
[ERROR] /home/mpom/Downloads/../Documents/android-testdpc/app/src/main/java/com/afwsamples/testdpc/policy/PolicyManagementFragment.java:675:5: Cyclomatic Complexity is 100 (max allowed is 10). [CyclomaticComplexity]

```
 @Override
    @TargetApi(Build.VERSION_CODES.N)
    public boolean onPreferenceClick(Preference preference) {
        String key = preference.getKey();
        switch (key) {
            case MANAGE_LOCK_TASK_LIST_KEY:
                showManageLockTaskListPrompt(R.string.lock_task_title,
                        new ManageLockTaskListCallback() {
                            @Override
                            public void onPositiveButtonClicked(String[] lockTaskArray) {
                                try {
                                    mDevicePolicyManager.setLockTaskPackages(
                                            DeviceAdminReceiver.getComponentName(getActivity()),
                                            lockTaskArray);
                                } catch (SecurityException e) {
                                    Log.d(TAG, "Exception when setting lock task packages", e);
                                    showToast(R.string.lock_task_unavailable);
                                }
                            }
                        }
                );
                return true;
            case CHECK_LOCK_TASK_PERMITTED_KEY:
                showCheckLockTaskPermittedPrompt();
                return true;
            case SET_LOCK_TASK_FEATURES_KEY:
                showFragment(new SetLockTaskFeaturesFragment());
                return true;
            case RESET_PASSWORD_KEY:
                if (BuildCompat.isAtLeastO()) {
                    showFragment(new ResetPasswordWithTokenFragment());
                    return true;
                } else {
                    showResetPasswordPrompt();
                    return false;
                }
            case LOCK_NOW_KEY:
                lockNow();
                return true;
            case START_LOCK_TASK:
                // Uses {@link Activity#startLockTask}
                getActivity().startLockTask();
                return true;
            case RELAUNCH_IN_LOCK_TASK:
                // Uses {@link ActivityOptions#setLockTaskMode}
                relaunchInLockTaskMode();
                return true;
            case STOP_LOCK_TASK:
                try {
                    getActivity().stopLockTask();
                } catch (IllegalStateException e) {
                    // no lock task present, ignore
                }
                return true;
            case WIPE_DATA_KEY:
                showWipeDataPrompt();
                return true;
            case PERSISTENT_DEVICE_OWNER_KEY:
                showFragment(new PersistentDeviceOwnerFragment());
                return true;
            case REMOVE_DEVICE_OWNER_KEY:
                showRemoveDeviceOwnerPrompt();
                return true;
            case REQUEST_BUGREPORT_KEY:
                requestBugReport();
                return true;
            case REQUEST_NETWORK_LOGS:
                showFragment(new NetworkLogsFragment());
                return true;
            case REQUEST_SECURITY_LOGS:
                showFragment(new SecurityLogsFragment());
                return true;
            case SET_ACCESSIBILITY_SERVICES_KEY:
                // Avoid starting the same task twice.
                if (mGetAccessibilityServicesTask != null && !mGetAccessibilityServicesTask
                        .isCancelled()) {
                    mGetAccessibilityServicesTask.cancel(true);
                }
                mGetAccessibilityServicesTask = new GetAccessibilityServicesTask();
                mGetAccessibilityServicesTask.execute();
                return true;
            case SET_INPUT_METHODS_KEY:
                // Avoid starting the same task twice.
                if (mGetInputMethodsTask != null && !mGetInputMethodsTask.isCancelled()) {
                    mGetInputMethodsTask.cancel(true);
                }
                mGetInputMethodsTask = new GetInputMethodsTask();
                mGetInputMethodsTask.execute();
                return true;
            case SET_NOTIFICATION_LISTENERS_KEY:
                // Avoid starting the same task twice.
                if (mGetNotificationListenersTask != null
                        && !mGetNotificationListenersTask.isCancelled()) {
                    mGetNotificationListenersTask.cancel(true);
                }
                mGetNotificationListenersTask = new GetNotificationListenersTask();
                mGetNotificationListenersTask.execute();
                return true;
            case SET_NOTIFICATION_LISTENERS_TEXT_KEY:
                setNotificationWhitelistEditBox();
                return true;
            case SET_DISABLE_ACCOUNT_MANAGEMENT_KEY:
                showSetDisableAccountManagementPrompt();
                return true;
            case GET_DISABLE_ACCOUNT_MANAGEMENT_KEY:
                showDisableAccountTypeList();
                return true;
            case ADD_ACCOUNT_KEY:
                getActivity().startActivity(new Intent(getActivity(), AddAccountActivity.class));
                return true;
            case REMOVE_ACCOUNT_KEY:
                chooseAccount();
                return true;
            case CREATE_MANAGED_PROFILE_KEY:
                showSetupManagement();
                return true;
            case CREATE_AND_MANAGE_USER_KEY:
                showCreateAndManageUserPrompt();
                return true;
            case REMOVE_USER_KEY:
                showRemoveUserPrompt();
                return true;
            case SWITCH_USER_KEY:
                showSwitchUserPrompt();
                return true;
            case START_USER_IN_BACKGROUND_KEY:
                showStartUserInBackgroundPrompt();
                return true;
            case STOP_USER_KEY:
                showStopUserPrompt();
                return true;
            case LOGOUT_USER_KEY:
                logoutUser();
                return true;
            case SET_USER_SESSION_MESSAGE_KEY:
                showFragment(new SetUserSessionMessageFragment());
                return true;
            case SET_AFFILIATION_IDS_KEY:
                showFragment(new ManageAffiliationIdsFragment());
                return true;
            case BLOCK_UNINSTALLATION_BY_PKG_KEY:
                showBlockUninstallationByPackageNamePrompt();
                return true;
            case BLOCK_UNINSTALLATION_LIST_KEY:
                showBlockUninstallationPrompt();
                return true;
            case ENABLE_SYSTEM_APPS_KEY:
                showEnableSystemAppsPrompt();
                return true;
            case ENABLE_SYSTEM_APPS_BY_PACKAGE_NAME_KEY:
                showEnableSystemAppByPackageNamePrompt();
                return true;
            case ENABLE_SYSTEM_APPS_BY_INTENT_KEY:
                showFragment(new EnableSystemAppsByIntentFragment());
                return true;
            case INSTALL_EXISTING_PACKAGE_KEY:
                showInstallExistingPackagePrompt();
                return true;
            case HIDE_APPS_KEY:
                showHideAppsPrompt(false);
                return true;
            case UNHIDE_APPS_KEY:
                showHideAppsPrompt(true);
                return true;
            case SUSPEND_APPS_KEY:
                showSuspendAppsPrompt(false);
                return true;
            case UNSUSPEND_APPS_KEY:
                showSuspendAppsPrompt(true);
                return true;
            case CLEAR_APP_DATA_KEY:
                showClearAppDataPrompt();
                return true;
            case KEEP_UNINSTALLED_PACKAGES:
                showFragment(new ManageKeepUninstalledPackagesFragment());
                return true;
            case MANAGE_APP_RESTRICTIONS_KEY:
                showFragment(new ManageAppRestrictionsFragment());
                return true;
            case DISABLE_METERED_DATA_KEY:
                showSetMeteredDataPrompt();
                return true;
            case GENERIC_DELEGATION_KEY:
                showFragment(new DelegationFragment());
                return true;
            case APP_RESTRICTIONS_MANAGING_PACKAGE_KEY:
                showFragment(new AppRestrictionsManagingPackageFragment());
                return true;
            case SET_PERMISSION_POLICY_KEY:
                showSetPermissionPolicyDialog();
                return true;
            case MANAGE_APP_PERMISSIONS_KEY:
                showFragment(new ManageAppPermissionsFragment());
                return true;
            case INSTALL_KEY_CERTIFICATE_KEY:
                Util.showFileViewerForImportingCertificate(this,
                        INSTALL_KEY_CERTIFICATE_REQUEST_CODE);
                return true;
            case REMOVE_KEY_CERTIFICATE_KEY:
                choosePrivateKeyForRemoval();
                return true;
            case GENERATE_KEY_CERTIFICATE_KEY:
                showPromptForGeneratedKeyAlias("generated-rsa-testdpc-1");
                return true;
            case TEST_KEY_USABILITY_KEY:
                testKeyCanBeUsedForSigning();
                return true;
            case INSTALL_CA_CERTIFICATE_KEY:
                Util.showFileViewerForImportingCertificate(this,
                        INSTALL_CA_CERTIFICATE_REQUEST_CODE);
                return true;
            case GET_CA_CERTIFICATES_KEY:
                showCaCertificateList();
                return true;
            case REMOVE_ALL_CERTIFICATES_KEY:
                mDevicePolicyManager.uninstallAllUserCaCerts(mAdminComponentName);
                showToast(R.string.all_ca_certificates_removed);
                return true;
            case MANAGED_PROFILE_SPECIFIC_POLICIES_KEY:
                showFragment(new ProfilePolicyManagementFragment(),
                        ProfilePolicyManagementFragment.FRAGMENT_TAG);
                return true;
            case LOCK_SCREEN_POLICY_KEY:
                showFragment(new LockScreenPolicyFragment.Container());
                return true;
            case PASSWORD_CONSTRAINTS_KEY:
                showFragment(new PasswordConstraintsFragment.Container());
                return true;
            case SYSTEM_UPDATE_POLICY_KEY:
                showFragment(new SystemUpdatePolicyFragment());
                return true;
            case SYSTEM_UPDATE_PENDING_KEY:
                showPendingSystemUpdate();
                return true;
            case SET_ALWAYS_ON_VPN_KEY:
                showFragment(new AlwaysOnVpnFragment());
                return true;
            case SET_GLOBAL_HTTP_PROXY_KEY:
                showSetGlobalHttpProxyDialog();
                return true;
            case CLEAR_GLOBAL_HTTP_PROXY_KEY:
                mDevicePolicyManager.setRecommendedGlobalProxy(mAdminComponentName,
                        null /* proxyInfo */);
                return true;
            case NETWORK_STATS_KEY:
                showFragment(new NetworkUsageStatsFragment());
                return true;
            case DELEGATED_CERT_INSTALLER_KEY:
                showFragment(new DelegatedCertInstallerFragment());
                return true;
            case DISABLE_STATUS_BAR:
                setStatusBarDisabled(true);
                return true;
            case REENABLE_STATUS_BAR:
                setStatusBarDisabled(false);
                return true;
            case DISABLE_KEYGUARD:
                setKeyGuardDisabled(true);
                return true;
            case REENABLE_KEYGUARD:
                setKeyGuardDisabled(false);
                return true;
            case START_KIOSK_MODE:
                showManageLockTaskListPrompt(R.string.kiosk_select_title,
                        new ManageLockTaskListCallback() {
                            @Override
                            public void onPositiveButtonClicked(String[] lockTaskArray) {
                                startKioskMode(lockTaskArray);
                            }
                        }
                );
                return true;
            case CAPTURE_IMAGE_KEY:
                dispatchCaptureIntent(MediaStore.ACTION_IMAGE_CAPTURE,
                        CAPTURE_IMAGE_REQUEST_CODE, mImageUri);
                return true;
            case CAPTURE_VIDEO_KEY:
                dispatchCaptureIntent(MediaStore.ACTION_VIDEO_CAPTURE,
                        CAPTURE_VIDEO_REQUEST_CODE, mVideoUri);
                return true;
            case CREATE_WIFI_CONFIGURATION_KEY:
                showWifiConfigCreationDialog();
                return true;
            case CREATE_EAP_TLS_WIFI_CONFIGURATION_KEY:
                showEapTlsWifiConfigCreationDialog();
                return true;
            case MODIFY_WIFI_CONFIGURATION_KEY:
                showFragment(new WifiModificationFragment());
                return true;
            case SHOW_WIFI_MAC_ADDRESS_KEY:
                showWifiMacAddress();
                return true;
            case SET_USER_RESTRICTIONS_KEY:
                showFragment(new UserRestrictionsDisplayFragment());
                return true;
            case REBOOT_KEY:
                reboot();
                return true;
            case SET_SHORT_SUPPORT_MESSAGE_KEY:
                showFragment(SetSupportMessageFragment.newInstance(
                        SetSupportMessageFragment.TYPE_SHORT));
                return true;
            case SET_LONG_SUPPORT_MESSAGE_KEY:
                showFragment(SetSupportMessageFragment.newInstance(
                        SetSupportMessageFragment.TYPE_LONG));
                return true;
            case SET_NEW_PASSWORD:
                startActivity(new Intent(DevicePolicyManager.ACTION_SET_NEW_PASSWORD));
                return true;
            case SET_PROFILE_PARENT_NEW_PASSWORD:
                startActivity(
                        new Intent(DevicePolicyManager.ACTION_SET_NEW_PARENT_PROFILE_PASSWORD));
                return true;
            case BIND_DEVICE_ADMIN_POLICIES:
                showFragment(new BindDeviceAdminFragment());
                return true;
            case CROSS_PROFILE_APPS:
                showFragment(new CrossProfileAppsFragment());
                return true;
            case SET_SCREEN_BRIGHTNESS_KEY:
                showSetScreenBrightnessDialog();
                return true;
            case SET_SCREEN_OFF_TIMEOUT_KEY:
                showSetScreenOffTimeoutDialog();
                return true;
            case TRANSFER_OWNERSHIP_KEY:
                showFragment(new PickTransferComponentFragment());
                return true;
            case SET_TIME_KEY:
                // Disable auto time before we could set time manually.
                mDevicePolicyManager.setGlobalSetting(mAdminComponentName,
                        Settings.Global.AUTO_TIME, "0");
                showSetTimeDialog();
                return true;
            case SET_TIME_ZONE_KEY:
                mDevicePolicyManager.setGlobalSetting(mAdminComponentName,
                        Settings.Global.AUTO_TIME_ZONE, "0");
                showSetTimeZoneDialog();
                return true;
            case MANAGE_OVERRIDE_APN_KEY:
                showFragment(new OverrideApnFragment());
                return true;
        }
        return false;
}
```
