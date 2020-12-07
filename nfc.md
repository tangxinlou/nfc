/*******************************************************
$Author       : 唐新楼 
$Last modified: 2020-12-07 16:32
$Email        : tangxinlou@wintech.com
$blog         : https://blog.csdn.net/daweigongting
$Filename     : nfc.md
$Description  : tangxinlou
*******************************************************/


```c

/d/work/nfc/NFC_NCIHAL_Nfc/NFC_NCIHAL_Nfc-NFC_AR_00_18C0_11.03.00_OpnSrc/src/com/android/nfc/NfcService.java:1023:
        boolean enableInternal() {
            synchronized (NfcService.this){
                if (mState == NfcAdapter.STATE_ON || mState == NfcAdapter.STATE_TURNING_ON) {
                    return true;
                }
                if(mState == NfcAdapter.STATE_TURNING_OFF) {
                    return false;
                }
                Log.i(TAG, "Enabling NFC");
                NfcStatsLog.write(NfcStatsLog.NFC_STATE_CHANGED, NfcStatsLog.NFC_STATE_CHANGED__STATE__ON);
                updateState(NfcAdapter.STATE_TURNING_ON);
            }

            WatchDogThread watchDog = new WatchDogThread("enableInternal", INIT_WATCHDOG_MS);
            watchDog.start();
            try {
                mRoutingWakeLock.acquire();
                try {
                    if (!mDeviceHost.initialize()) {
                        Log.w(TAG, "Error enabling NFC");
                        updateState(NfcAdapter.STATE_OFF);
                        return false;
                    }
                } finally {
                    mRoutingWakeLock.release();
                }
            } finally {
                watchDog.cancel();
            }
            int uiccSlot = 0;
            uiccSlot = mPrefs.getInt(PREF_CUR_SELECTED_UICC_ID, SECURE_ELEMENT_UICC_SLOT_DEFAULT);
            mDeviceHost.setPreferredSimSlot(uiccSlot);
            mOverflowDefaultRoute = ROUTE_INVALID;
            if (mIsHceCapable) {
                // Generate the initial card emulation routing table
                mCardEmulationManager.onNfcEnabled();
                computeRoutingParameters();
            }

            nci_version = getNciVersion();
            Log.d(TAG, "NCI_Version: " + nci_version);

            synchronized (NfcService.this) {
                mObjectMap.clear();
                if (mIsBeamCapable) {
                    mP2pLinkManager.enableDisable(mIsNdefPushEnabled, true);
                }
                onPreferredPaymentChanged(NfcAdapter.PREFERRED_PAYMENT_LOADED);
            }

            initSoundPool();

            mScreenState = mScreenStateHelper.checkScreenState();
            int screen_state_mask = (mNfcUnlockManager.isLockscreenPollingEnabled()) ?
                             (ScreenStateHelper.SCREEN_POLLING_TAG_MASK | mScreenState) : mScreenState;

            if(mNfcUnlockManager.isLockscreenPollingEnabled())
                applyRouting(false, false);

            mDeviceHost.doSetScreenState(screen_state_mask);
            mPreviousScreenState = mScreenState;

            sToast_debounce = false;

            /* Start polling loop */
            applyRouting(true, false);
            commitRouting();
            /* WiredSe Init after ESE is discovered and initialised */
            initWiredSe();
            synchronized (NfcService.this) {
                updateState(NfcAdapter.STATE_ON);
            }
            return true;
        }

```

































