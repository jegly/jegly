<p align="center">
  <img src="https://raw.githubusercontent.com/jegly/jegly/main/jes.gif" alt="Chaotic Cubes" width="250">
</p>

# My experience with VRP, Google assigned 30+ of my zero-click Android bugs, had me sign a CLA, then closed them "duplicate" or "not a vulnerability" and shipped the fixes. Eight months on, a bounty they confirmed in writing is still half unpaid. 85+ total bugs submitted over 8 months of everyday work.

This repository documents a pattern across my own Android VRP reports to Google. Everything below concerns reports Google has already **closed** (as duplicate, infeasible, can't-repro, or working-as-intended). Nothing here concerns a report that is still open or under active coordinated disclosure. Where I quote Google, the quote is verbatim from the ticket.

## Summary

I have spent most of the last eight months doing security research on a current Pixel, reverse engineering the Samsung radio-interface binaries, the IMS and RTP native stack, the Trusty trusted apps, and the telephony framework. Over thirty distinct findings, most with a working proof of concept, several confirmed with AddressSanitizer against real compiled code.

Here is what I have to show for it. One bounty, for two older bugs, that Google confirmed in writing at five hundred dollars total. Half of it, two hundred fifty dollars, has been paid. The other half has been "processing" since January 2026. That is over eight months of "manual exception workflow" and "we cannot give you an exact date."

This is not one unlucky ticket, and it is two separate problems, not one. Going through Google's own current factory images function by function, I can confirm **8 of my closed findings were fixed in the shipped code with no reward or credit at all**. That is the money and credit problem. Separately, **14 more were closed as duplicate, infeasible, or can't-repro, and the exact defect I reported is still unchanged in the current build**. Nobody owes a reward for an unfixed bug. What that second group proves is different: that the stated reason for closing them does not hold up against the actual code. Both tables below list these by ticket number, kept apart on purpose.

## Methodology

For every bug in the case index below, the verdict was reached the same way:

1. Pull the exact binary, library, or jar that the original report named, out of Google's current factory image (rango and yogi, Pixel builds dated September 2026).
2. Extract it with the correct tool for its container: `debugfs` for ext4 partitions, `fsck.erofs` for EROFS, double extraction through `apex_payload.img` for APEX modules.
3. Locate the exact function the report cited, using symbol resolution where the binary still has symbols, or byte-pattern matching against the report's own quoted disassembly where it does not.
4. Compare the current code against what the report described as vulnerable.

I did not use Google's own security bulletin or the supplemental patch list as a substitute for this. A silent fix with no bulletin entry and no credit would not show up there. The only way to find one is to diff the actual shipped code.

## The reward they promised me in writing, and still have not paid

Before the thirty-plus newer bugs, there were two older ones. On ticket **467575499**, on January 23rd 2026, Google wrote that they had "confirmed this is a valid bug," were "increasing severity," "increasing reward," and would "issue a payment for the new amount once it has been processed." Three months later, on April 24th, with no new analysis and no new evidence, they reversed all three determinations. The same bug now "does not qualify for a reward or severity increase," called a "Final Assessment."

Then, in June, they wrote that they wanted to "stand by our word regarding the reward." In July, on the 10th, they confirmed the number directly: "$250 for Safety Center, $250 for the mic bug, $500 total," and said they were "honoring our initial promise," even while classifying the Safety Center half as a duplicate with no security impact. The mic-bug half was paid. The Safety Center half has been sitting in a "discretionary payment exception" that Google "cannot give an exact credit date" for, since January. That is a debt confirmed in their own writing, on the record, unpaid for over eight months.

## The admitted duplicate-ID collision

Google's basis for closing a report as a duplicate is an internal canonical bug ID. I asked them to produce the specific numbers behind several of these closures.

They cited the exact same internal canonical ID, **479203197**, as the original report for two completely unrelated bugs of mine: a Rust keystore panic in a trusted app, and a C++ out-of-bounds read in the RTP media parser. Different languages, different subsystems, different teams. One internal report cannot be the root cause of both.

Google admitted the error in writing, on both tickets independently: "Giving you this ID for your other report was a mistake on our end, and we apologise for the confusion," and separately, "we made a mistake when linking the duplicate tickets." That means at least one "verified duplicate" was stamped without anyone actually checking it against the report it supposedly matched.

## They have used the exact "not a vulnerability" template before, and retracted it

Earlier in this campaign, Google closed one finding with the boilerplate "not a security vulnerability... logged for potential remediation in a future version... considered closed." They later reversed it in writing: "You are completely correct. This finding is indeed a legitimate security vulnerability... it was incorrectly closed as 'not a security vulnerability' due to an oversight during triage of this specific ticket."

The same template that closed several more of my bugs afterward has a documented track record, on my own tickets, of being wrong by Google's own admission.

## Case index: confirmed fixed, closed with no reward

Every row below is a **closed** report, checked directly against the current September 2026 factory image. "Fixed" means the specific code the report described changed; the report was correct, the fix shipped, and the ticket was still closed with no reward.

| Ticket | Component / function | Closure reason | Verdict |
|---|---|---|---|
| 549662989 | PixelModemService `MintReceiver` | Duplicate | Fixed. The guard permission `MINT_CONFIGURATION_UPDATE_BROADCAST` is now defined, signature\|privileged. |
| 538369053 | framework.jar `PduParser.parseParts` | Duplicate, "previously reported by an internal Google engineer" | Fixed. A bound check against `stream.available()` was added before the allocation that used to run unconditionally. |
| 560866996 | framework.jar `PduParser` (same defect as above) | Duplicate, canonical 513581684, "internal Google engineer" | Fixed, same underlying code change as 538369053. |
| 545948346 | libpixelimsmedia.so `RtpSession::numberOfReportBlocks` | Duplicate, "previously reported by an internal Google engineer" | Fixed. An underflow check was added before the division that used to produce garbage. |
| 540755547 | libpixelimsmedia.so `RtcpChunk::decodeRtcpChunk` | Duplicate | Fixed. A bound check on the SDES item length was added before the allocation and copy. |
| 541096815 | libpixelimsmedia.so `RtpDecoderNode::DecodeRtpHeaderExtension` | Infeasible, "not a security vulnerability" | Fixed. The length byte is now read unsigned instead of signed, with a new bound check added after it. |
| 540753210 | libpixelimsmedia.so `RtcpFbPacket::decodeRtcpFbPacket` | Duplicate | Fixed. A length check was added before the operation that previously underflowed unconditionally. |
| 529518244 | ShannonRcs.apk `DeviceProvisioningService` | Duplicate | Fixed. The service now requires a signature\|privileged permission where none existed before. |

That is 8 findings, each with its own ticket number, that Google closed as duplicate or infeasible and then fixed in the shipping image.

## Case index: still unpatched, closed with reasoning that does not hold up

This table is not a payment claim. None of these bugs were fixed, so no reward is owed for them. What this table shows is that the stated reason for closing each one does not match the actual code: the defect described in the original report is still there, unchanged, in the current factory image. Every row is a **closed** report.

| Ticket | Component / function | Closure reason | Verdict |
|---|---|---|---|
| 561846309 | ConfigUpdater.apk `PhenotypeHelper.getCertificate()` | WAI | Unchanged. Returns one certificate for every config type while its two sibling methods correctly switch on type. |
| 552305734 | CrossDeviceAccessServicePrimary.apk `PermissionBroadcastReceiver` | Infeasible | Unchanged. Exported, no permission. |
| 558478942 | DCMO.apk `DcmoReceiver` | Duplicate | Unchanged. Exported, no permission, matches its unguarded siblings. |
| 555981160 | telephony-common.jar `ValueParser.retrieveItemsIconId` | Can't Repro | Unchanged. No lower-bound check before `new-array` on a decremented length. |
| 538367541 / 543174762 | telephony-common.jar `WapPushOverSms.decodeWapPdu` | Duplicate | Unchanged. Unvalidated decoded uintvar used directly as an allocation size. |
| 557568085 | TelephonyProvider.apk `updateSynchronized()` | Duplicate | Unchanged. Caller-supplied selection string concatenated with no parentheses before the ownership clause. |
| 560453159 | service-ranging.jar `RangingServiceImpl` (UWB Digital Key) | Can't Repro | Unchanged on the stable image. The same fix exists on the Android 17 QPR2 beta track, raising the question of which build was actually tested. |
| 555984702 | grilservice.apk `RadioConfig.readFromParcel` | Can't Repro | Unchanged. Raw `readInt()` size used directly as an allocation with no upper bound. |
| 552188030 | services.jar `Vpn.setVpnForcedLocked()` | Duplicate | Unchanged. UID 0 is excluded from the VPN lockdown range. |
| 558472707 | service-npumanager.jar `NpuManagerServiceImpl.cancelModelLoad` | Infeasible | Unchanged. No caller-identity check, unlike sibling methods on the same class. |
| 537920968 | nfc_nci.st21nfc.default.so `handlePollingLoopData` | Infeasible | Unchanged. Same undersized length-check logic, confirmed at the function's new address after a build-layout shift. |
| 538311587 | android.hardware.secure_element-service.uicc | Infeasible | Unchanged. Same mutex/slot/type offsets, no per-call correlation token. |
| 538314136 | IntentResolver.apk `UriMetadataReaderImpl.getMetadata` (DISPLAY_ICON_URI) | Infeasible | Unchanged. No ownership check on the cross-profile URI read. |
| 560878423 | libimsstack.so `SipMessageFraming::ParseMessageBody` | Can't Repro | Unchanged. A negative content length still passes a signed comparison into the completed-message state instead of being rejected. |

## This is not just me

In June 2026, The Register ran "Google told researcher 'Nice catch!' Then denied bug bounty for flaw it still hasn't fixed." Praise, then denial, then an unfixed bug. A separate researcher documented their own Google VRP case moving from "not a bug" to "not a vulnerability" to, finally, duplicate, meaning Google knew the whole time it was real. CSO Online has reported that legal experts believe these programs may raise labor-law questions, treating people who do the work of employees as disposable contractors whose finished work can be rejected with no explanation and no recourse.

I am not against Google generally. I fork and contribute to their open source projects. This is specifically about what happened when I reported real, working security findings through their own stated process.

