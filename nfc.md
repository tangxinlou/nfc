/*******************************************************
$Author       : 唐新楼 
$Last modified: 2020-12-07 16:32
$Email        : tangxinlou@wintech.com
$blog         : https://blog.csdn.net/daweigongting
$Filename     : nfc.md
$Description  : tangxinlou
*******************************************************/


```c

/d/work/nfc/NFC_NCIHAL_base/core/java/android/nfc/NfcAdapter.java:883:
public boolean enable() {
        try {
            return sService.enable();
        } catch (RemoteException e) {
            attemptDeadServiceRecovery(e);
            return false;
        }
    }

```


```c

/d/work/nfc/NFC_NCIHAL_Nfc/src/com/android/nfc/NfcService.java:1231:
final class NfcAdapterService extends INfcAdapter.Stub {
        @Override
        public boolean enable() throws RemoteException {

            NfcPermissions.enforceAdminPermissions(mContext);

            saveNfcOnSetting(true);

            new EnableDisableTask().execute(TASK_ENABLE);

            return true;
        }

```


```c

 /d/work/nfc/NFC_NCIHAL_Nfc/src/com/android/nfc/NfcService.java:963
     class EnableDisableTask extends AsyncTask<Integer, Void, Void>  

```

```c

/d/work/nfc/NFC_NCIHAL_Nfc/src/com/android/nfc/NfcService.java:1023:
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


```c

/d/work/nfc/NFC_NCIHAL_Nfc/nci/src/com/android/nfc/dhimpl/NativeNfcManager.java:119:
public boolean initialize() {
        boolean ret = doInitialize();
        mIsoDepMaxTransceiveLength = getIsoDepMaxTransceiveLength();
        return ret;
    }

```


```c

 /d/work/nfc/NFC_NCIHAL_Nfc/nci/jni/NativeNfcManager.cp:4350:
 {"doInitialize", "()Z", (void*)nfcManager_doInitialize} 

```


```c

/d/work/nfc/NFC_NCIHAL_Nfc/nci/jni/NativeNfcManager.cpp:2068:
static jboolean nfcManager_doInitialize(JNIEnv * e, jobject o) {
    initializeGlobalDebugEnabledFlag();
    tNFA_MW_VERSION mwVer;
    gSeDiscoverycount = 0;
    gActualSeCount = 0;

    uint8_t switchToUiccSlot = 0;
#if (NXP_EXTNS == TRUE)
    rfActivation = false;
    tNFA_PMID ven_config_addr[] = {0xA0, 0x07};
    tNFA_PMID pollProfileAddr[] = {0xA0, 0x44};
    uint8_t pollProfileVal[] = {NFC_FORUM_POLL};
    bool isSuccess = false;
    sNfcee_disc_state = UICC_SESSION_NOT_INTIALIZED;
    IsEseCeDisabled = false;
    /* NFC initialization in progress */
    if (NFC_OFF == sNfcState) sNfcState = NFC_INITIALIZING_IN_PROGRESS;

    char propBuf[PROPERTY_VALUE_MAX] = {'\0'};
    int len = property_get(ESE_PWR_MGMT_PROP, propBuf, "");
    if (len != 0) {
      sIsLowRamDevice = (propBuf[0] != '\0')
                            ? ((strcmp(propBuf, "true") == 0) ? true : false)
                            : true;
      DLOG_IF(INFO, nfc_debug_enabled) << StringPrintf(
          "isLowRamDevice %s", sIsLowRamDevice ? "true" : "false");
    }
    NFA_SetLowRamDevice(sIsLowRamDevice);
#endif
    mwVer = NFA_GetMwVersion();
    DLOG_IF(INFO, nfc_debug_enabled)
        << StringPrintf("%s:  MW Version: NFC_AR_%02X_%04x_%02d.%02x.%02x", __func__,
                        mwVer.cust_id, mwVer.validation, mwVer.android_version,
                        mwVer.major_version, mwVer.minor_version);

    tNFA_STATUS stat = NFA_STATUS_OK;
    NfcTag::getInstance().mNfcDisableinProgress = false;
    PowerSwitch& powerSwitch = PowerSwitch::getInstance();
    if (sIsNfaEnabled) {
      DLOG_IF(INFO, nfc_debug_enabled)
          << StringPrintf("%s: already enabled", __func__);
      goto TheEnd;
    }
#if (NXP_EXTNS == TRUE)
    if (gsNfaPartialEnabled) {
      DLOG_IF(INFO, nfc_debug_enabled) << StringPrintf(
          "%s: already  partial enable calling deinitialize", __func__);
      nfcManager_doPartialDeInitialize();
    }
#endif
    powerSwitch.initialize(PowerSwitch::FULL_POWER);
    register_signal_handler();
    {

      NfcAdaptation& theInstance = NfcAdaptation::GetInstance();
      theInstance.Initialize();  // start GKI, NCI task, NFC task
#if (NXP_EXTNS == TRUE)
      int state = getJCOPOS_UpdaterState();
      if ((state != OSU_COMPLETE) && (state != OSU_NOT_STARTED)) {
        DLOG_IF(INFO, nfc_debug_enabled) << StringPrintf("JCOP is in OSU mode");
        NFA_SetBootMode(NFA_OSU_BOOT_MODE);
      } else {
        NFA_SetBootMode(NFA_NORMAL_BOOT_MODE);
      }
#endif
      stat = nfcManagerEnableNfc(theInstance);
      nfcManager_getFeatureList();
      EXTNS_Init(nfaDeviceManagementCallback, nfaConnectionCallback);

      if (stat == NFA_STATUS_OK) {
        // sIsNfaEnabled indicates whether stack started successfully
        if (sIsNfaEnabled) {
          SecureElement::getInstance().initialize(getNative(e, o));
          sRoutingInitialized =
              RoutingManager::getInstance().initialize(getNative(e, o));
          HciRFParams::getInstance().initialize();
          MposManager::getInstance().initialize(getNative(e, o));
          sIsSecElemSelected =
              (SecureElement::getInstance().getActualNumEe() - 1);
          sIsSecElemDetected = sIsSecElemSelected;
          nativeNfcTag_registerNdefTypeHandler();
          NfcTag::getInstance().initialize(getNative(e, o));
          PeerToPeer::getInstance().initialize();
          PeerToPeer::getInstance().handleNfcOnOff(true);
          HciEventManager::getInstance().initialize(getNative(e, o));
#if (NXP_EXTNS == TRUE)
          if (NfcConfig::hasKey(NAME_NXP_DEFAULT_NFCEE_DISC_TIMEOUT)) {
            gdisc_timeout =
                NfcConfig::getUnsigned(NAME_NXP_DEFAULT_NFCEE_DISC_TIMEOUT);
          } else {
            DLOG_IF(INFO, nfc_debug_enabled) << StringPrintf(
                "NAME_NXP_DEFAULT_NFCEE_DISC_TIMEOUT not found");
            gdisc_timeout =
                NFCEE_DISC_TIMEOUT_SEC; /*Default nfcee discover timeout*/
          }
          gdisc_timeout = gdisc_timeout * 1000;
          if (NFA_STATUS_OK == GetNumNFCEEConfigured()) {
            DLOG_IF(INFO, nfc_debug_enabled)
                << StringPrintf(" gSeDiscoverycount = %d gActualSeCount=%d",
                                gSeDiscoverycount, gActualSeCount);
            if (gSeDiscoverycount < gActualSeCount) {
              DLOG_IF(INFO, nfc_debug_enabled) << StringPrintf(
                  "Wait for ESE to discover, gdisc_timeout = %d",
                  gdisc_timeout);
              SyncEventGuard g(gNfceeDiscCbEvent);
              if (gNfceeDiscCbEvent.wait(gdisc_timeout) == false) {
                LOG(ERROR) << StringPrintf(
                    "%s: timeout waiting for nfcee dis event", __func__);
              }
              DLOG_IF(INFO, nfc_debug_enabled)
                  << StringPrintf("gSeDiscoverycount  = %d gActualSeCount=%d",
                                  gSeDiscoverycount, gActualSeCount);
            } else {
              DLOG_IF(INFO, nfc_debug_enabled)
                  << StringPrintf("All ESE are discovered ");
            }
          }
          // Create transaction controller
          (void)transactionController::controller();
          if (nfcFL.nfccFL._NFC_NXP_STAT_DUAL_UICC_EXT_SWITCH) {
            checkforESERemoval();

            if (NfcConfig::hasKey(NAME_NXP_DUAL_UICC_ENABLE)) {
              dualUiccInfo.dualUiccEnable =
                  NfcConfig::getUnsigned(NAME_NXP_DUAL_UICC_ENABLE);
            }
            if (dualUiccInfo.dualUiccEnable == 0x01) {
              checkforNfceeConfig(UICC1 | UICC2 | ESE);
              dualUiccInfo.uiccActivStat = 0x00;
              if (SecureElement::getInstance().getEeStatus(
                      SecureElement::getInstance().EE_HANDLE_0xF4) !=
                  NFC_NFCEE_STATUS_REMOVED) {
                dualUiccInfo.uiccActivStat = (sSelectedUicc & 0x0F);
              }
              switchToUiccSlot = ((sSelectedUicc & 0x0F) == 0x01) ? 0x02 : 0x01;
              nfcManager_doSelectUicc(e, o, switchToUiccSlot);
              if (SecureElement::getInstance().getEeStatus(
                      SecureElement::getInstance().EE_HANDLE_0xF4) !=
                  NFC_NFCEE_STATUS_REMOVED) {
                dualUiccInfo.uiccActivStat |= (sSelectedUicc & 0x0F);
              }
            }
          } else
            SecureElement::getInstance().updateEEStatus();
          if (nfcFL.eseFL._JCOP_WA_ENABLE) {
            RoutingManager::getInstance().handleSERemovedNtf();
          }
          DLOG_IF(INFO, nfc_debug_enabled)
              << StringPrintf("Discovered se count %d", gSeDiscoverycount);
          /*Check for ETSI12 Configuration for SEs detected in the HCI Network*/
          performNfceeETSI12Config();
          if (nfcFL.eseFL._ESE_ETSI12_PROP_INIT &&
              (swp_getconfig_status & SWP2_ESE)) {
            performHCIInitialization(e, o);
          }
          SecureElement::getInstance().getSETechnology(
              SecureElement::EE_HANDLE_0xF3);
          checkforNfceeConfig(UICC1 | UICC2 | ESE);
          /*Pending clear all pipe handling*/
          if (sNfcee_disc_state == UICC_CLEAR_ALL_PIPE_NTF_RECEIVED) {
            DLOG_IF(INFO, nfc_debug_enabled)
                << StringPrintf("Perform pending UICC clear all pipe handling");
            sNfcee_disc_state = UICC_SESSION_INTIALIZATION_DONE;
            /*Handle UICC clear all pipe notification*/
            checkforNfceeConfig(UICC1 | UICC2);
          }
          sNfcee_disc_state = UICC_SESSION_INTIALIZATION_DONE;
#endif
/////////////////////////////////////////////////////////////////////////////////
// Add extra configuration here (work-arounds, etc.)
#if (NXP_EXTNS == TRUE)
          if (nfcFL.nfcNxpEse) {
            if (nfcFL.eseFL._ESE_SVDD_SYNC ||
                nfcFL.eseFL._ESE_JCOP_DWNLD_PROTECTION ||
                nfcFL.nfccFL._NFCC_SPI_FW_DOWNLOAD_SYNC ||
                nfcFL.eseFL._ESE_DWP_SPI_SYNC_ENABLE) {
              isSuccess = createSPIEvtHandlerThread();
            }
            if (nfcFL.eseFL._ESE_DWP_SPI_SYNC_ENABLE && isSuccess) {
              SecureElement::getInstance().enableDwp();
            } else if (!isSuccess) {
              DLOG_IF(INFO, nfc_debug_enabled)
                  << StringPrintf("Failed to start SPI Event Handler Thread");
            }
          }
#endif
          pendingScreenState = false;
          {
            SyncEventGuard guard(android::sNfaGetConfigEvent);
            stat = NFA_GetConfig(0x01, ven_config_addr);
            if (stat == NFA_STATUS_OK) {
              android::sNfaGetConfigEvent.wait();
            }
            /*sCurrentConfigLen should be > 4 (num_tlv:1 + addr:2 + value:1) and
             *pos 4 gives the current eeprom value*/
            if ((sCurrentConfigLen > 4) && (sConfig[4] == 0x03)) {
              DLOG_IF(INFO, nfc_debug_enabled) << StringPrintf(
                  "%s: No need to update VEN_CONFIG. Already set to 0x%02x",
                  __func__, sConfig[4]);
            } else {
              SetVenConfigValue(NFC_MODE_ON);
              if (stat != NFA_STATUS_OK) {
                LOG(ERROR) << StringPrintf(
                    "%s: fail enable SetVenConfigValue; error=0x%X", __func__,
                    stat);
              }
            }
            gGeneralPowershutDown = 0;
          }
          updateEeprom(pollProfileAddr, sizeof(pollProfileVal), pollProfileVal);
          if (gIsDtaEnabled == true) {
            uint8_t configData = 0;
            configData =
                0x01; /**< Poll NFC-DEP : Highest Available Bit Rates */
            NFA_SetConfig(NCI_PARAM_ID_BITR_NFC_DEP, sizeof(uint8_t),
                          &configData);
            configData = 0x0B; /**< Listen NFC-DEP : Waiting Time */
            NFA_SetConfig(NFC_PMID_WT, sizeof(uint8_t), &configData);
            configData =
                0x0F; /**< Specific Parameters for NFC-DEP RF Interface */
            NFA_SetConfig(NCI_PARAM_ID_NFC_DEP_OP, sizeof(uint8_t),
                          &configData);
          }

        struct nfc_jni_native_data* nat = getNative(e, o);

          if (nat) {
          nat->tech_mask =
              NfcConfig::getUnsigned(NAME_POLLING_TECH_MASK, DEFAULT_TECH_MASK);
            DLOG_IF(INFO, nfc_debug_enabled) << StringPrintf(
                "%s: tag polling tech mask=0x%X", __func__, nat->tech_mask);
          }

          // if this value exists, set polling interval.
        nat->discovery_duration = NfcConfig::getUnsigned(
            NAME_NFA_DM_DISC_DURATION_POLL, DEFAULT_DISCOVERY_DURATION);
#if (NXP_EXTNS == TRUE)
          discDuration = nat->discovery_duration;
#endif
          NFA_SetRfDiscoveryDuration(nat->discovery_duration);

          // get LF_T3T_MAX
          {
            SyncEventGuard guard(sNfaGetConfigEvent);
            tNFA_PMID configParam[1] = {NCI_PARAM_ID_LF_T3T_MAX};
            stat = NFA_GetConfig(1, configParam);
            if (stat == NFA_STATUS_OK) {
              sNfaGetConfigEvent.wait();
              if (sCurrentConfigLen >= 4 ||
                  sConfig[1] == NCI_PARAM_ID_LF_T3T_MAX) {
                DLOG_IF(INFO, nfc_debug_enabled)
                    << StringPrintf("%s: lfT3tMax=%d", __func__, sConfig[3]);
                sLfT3tMax = sConfig[3];
              }
            }
          }
          unsigned long num = 0;
          if (NfcConfig::hasKey(NAME_NXP_CE_ROUTE_STRICT_DISABLE)) {
            num = NfcConfig::getUnsigned(NAME_NXP_CE_ROUTE_STRICT_DISABLE);
          } else {
            num = 0x01;
          }

          // TODO: Check this in L_OSP_EXT[PN547C2]
          //                NFA_SetCEStrictDisable(num);
          RoutingManager::getInstance().setCeRouteStrictDisable(num);
#if (NXP_EXTNS==TRUE)
          prevScreenState = NFA_SCREEN_STATE_UNKNOWN;
          StoreScreenState(NFA_SCREEN_STATE_ON_UNLOCKED);
#else
          prevScreenState = NFA_SCREEN_STATE_OFF_LOCKED;
#endif
#if (NXP_EXTNS != TRUE)
          // Do custom NFCA startup configuration.
          doStartupConfig();
#else
        if (HciRFParams::getInstance().isCeWithEseDisabled()) {
          recoverEseConnectivity();
        }
#endif
          goto TheEnd;
        }
      }

      LOG(ERROR) << StringPrintf("%s: fail nfa enable; error=0x%X", __func__,
                                 stat);

      if (sIsNfaEnabled) {
        EXTNS_Close();
        stat = NFA_Disable(false /* ungraceful */);
      }

      theInstance.Finalize();
    }

  TheEnd:
    if (sIsNfaEnabled)
      PowerSwitch::getInstance().setLevel(PowerSwitch::LOW_POWER);
    DLOG_IF(INFO, nfc_debug_enabled) << StringPrintf("%s: exit", __func__);
    return sIsNfaEnabled ? JNI_TRUE : JNI_FALSE;
  }

```

























