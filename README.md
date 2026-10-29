<p align="center">
  <img src="https://github.com/jegly/jegly/blob/main/jes.gif" alt="Chaotic Cubes" width="250">
</p>

# A year in Google's VRP: ~85 reports, one $250 payment, and the bugs they fixed after telling me they weren't real

For most of the last year this is what I've done. I reverse engineered the Samsung radio stack, the IMS and RTP media code, the Trusty trusted apps, the MediaTek M90 modem RIL, and the Android telephony framework on Google's current phones, and many more things. I filed roughly eighty six reports through the Android and Google Devices Vulnerability Reward Program. 95% had a working proof of concept. 

Total paid, for all of it: **$250. One payment.**

This isn't one unlucky ticket. The same thing keeps happening three different ways.

**They fixed my bugs after closing them.** I pulled each closed report's binary out of Google's current factory image and diffed the exact function the report named. Eight of them had been fixed in shipped code: a bounds check, a permission, a validation step added exactly where I said it was missing. Every one of those eight was first closed as a "duplicate" or "not a vulnerability." None was paid. None was credited. Two more have since been fixed on the QPR3 beta.

**They closed bugs with reasons the code contradicts.** More than a dozen others were closed duplicate, infeasible, or "can't reproduce," and the exact defect is still sitting unchanged in the current build. Nobody owes a reward for an unfixed bug, and I'm not claiming one. The point is narrower: the stated reason for closing them does not survive a look at the actual binary.

**They promised a reward in writing, in January, and still haven't paid it.** Google confirmed a valid bug, raised the severity, raised the reward, in writing. Three months later they reversed all three with no new evidence. Then they said they would "stand by our word" and pay it anyway as a $250 "discretionary payment exception." That was January. Ten months of "processing / soon / no date" later, it still isn't paid.

One line from that thread I still can't parse. They told me, in writing, that they are paying the Safety Center reward *"even though our final technical assessment rated it as a duplicate of a report with no security impact."* If it has no security impact there is no basis to pay it. If it is worth paying, "no security impact" is false. Both are in the ticket.

And the program rate limits anyone with fewer than five rewarded reports to three open at a time. My rewards kept getting closed or stalled, so for months I couldn't file the next batch. You can't get rewarded because you're rate limited, and you're rate limited because you haven't been rewarded.

## Where this stands right now

- **$250, Safety Center report (Moderate).** Approved in writing in January, reconfirmed in writing several times since. Unpaid ten months later. Ticket closed.
- **Two High severity reports, still "Assigned."** Both formally rated High by Google. Rewards undetermined, CVEs not assigned. One of the two is already fixed in Google's shipped code.
- **One Moderate report, still "Assigned."** Rated Moderate on 28 October. Reward undetermined.
- **Ten findings fixed after closure.** Closed as duplicate, infeasible, or not-a-vulnerability, then fixed in shipped code. No reward, no credit.
- **Total actually received across all of it: $250.**

Everything in the tables below is a **closed** report, with Google's own words quoted from the tracker. The three still "Assigned" reports and anything under active coordinated disclosure are kept out of the technical detail on purpose.

## How I checked

Same method for every closed report in the tables:

1. Pull the exact binary, library, or jar the report named, out of Google's current factory image (rango and yogi Pixel builds, September and October 2026).
2. Extract it with the right tool for the container: `debugfs` for ext4, `fsck.erofs` for EROFS, a second pass through `apex_payload.img` for APEX modules.
3. Find the exact function the report cited. Symbol resolution where symbols survive, byte pattern matching against the report's own quoted disassembly where they don't.
4. Diff the current shipped code against what the report called vulnerable, using the NDK `llvm-objdump`. (The system binutils has no AArch64 backend and returns nothing without an error, which is worth knowing before you trust an empty diff.)

Verdicts are **PATCHED** (a check was added that wasn't there when I reported it), **STILL PRESENT** (code unchanged), **N/A** (reported against this exact build, or Google called it intended design), and **CANT-DETERMINE** (binary absent or build mismatch).

I didn't use Google's bulletin or supplemental patch list for this. A silent fix with no bulletin entry and no credit doesn't appear there by definition. You have to diff the shipped code, which is what this is.

## The reward they promised in writing and still haven't paid

On ticket **467575499 / 470814942**, on **23 January 2026**, Google wrote that they had "confirmed this is a valid bug," were "increasing severity," "increasing reward," and would "issue a payment for the new amount once it has been processed." On **24 April**, with no new analysis, they reversed all three. The same bug now "does not qualify for a reward or severity increase," called a "Final Assessment."

In June they wrote that they wanted to "stand by our word regarding the reward." On **10 July** they gave the numbers: "$250 for Safety Center, $250 for the mic bug, $500 total," and said they were "honoring our initial promise," while still calling the Safety Center half a duplicate with no security impact. The mic half was paid. The Safety Center half went into a "discretionary payment exception" with, in their words, no "exact credit date." That was January. The most recent reply, in October, says the same thing: still approved, no date.

## The duplicate-ID collision they admitted

Google closes a report as a duplicate against an internal canonical bug ID. I asked for the specific IDs behind several of mine. They gave the **same internal ID, 479203197**, as the original for two unrelated bugs: a Rust keystore panic in a trusted app, and a C++ out-of-bounds read in the RTP media parser. Different languages, different subsystems, different teams. One internal report is not the root cause of both.

They admitted it in writing, on both tickets: "Giving you this ID for your other report was a mistake on our end," and separately, "we made a mistake when linking the duplicate tickets." So at least one "verified duplicate" was stamped without anyone checking it against the report it named.

## The "not a vulnerability" template, used before and retracted

Earlier in this, Google closed a finding with the standard "not a security vulnerability, logged for potential remediation in a future version, considered closed." They later took it back in writing: "You are completely correct. This finding is indeed a legitimate security vulnerability. It was incorrectly closed as 'not a security vulnerability' due to an oversight during triage of this specific ticket." The same template closed several more of my bugs after that. It has a record, on my own tickets, of being wrong by their own admission.

## Fixed in shipped code, closed with no reward

Every row is a closed report, diffed against Google's current factory image. "Fixed" means the code the report described changed: the report was right, the fix shipped, the ticket stayed closed, nothing was paid.

| Ticket | Component / function | Closure | Verdict |
|---|---|---|---|
| 549662989 | PixelModemService `MintReceiver` guard permission | Duplicate | Fixed. `MINT_CONFIGURATION_UPDATE_BROADCAST` now defined, signature\|privileged. |
| 538369053 | framework.jar `PduParser.parseParts` | Duplicate ("internal Google engineer") | Fixed. Bounds check against `stream.available()` added before the allocation. |
| 560866996 | framework.jar `PduParser` (same defect) | Duplicate, "internal Google engineer" | Fixed, same change as 538369053. |
| 545948346 | libpixelimsmedia.so `numberOfReportBlocks` | Duplicate, "internal Google engineer" | Fixed. Underflow check added before the division. |
| 540755547 | libpixelimsmedia.so `decodeRtcpChunk` | Duplicate | Fixed. Bounds check on the SDES item length before the copy. |
| 541096815 | libpixelimsmedia.so `DecodeRtpHeaderExtension` | Infeasible, "not a security vulnerability" | Fixed. Length byte now read unsigned, new bounds check after it. |
| 540753210 | libpixelimsmedia.so `decodeRtcpFbPacket` | Duplicate | Fixed. Length check added before the op that underflowed. |
| 529518244 | ShannonRcs.apk `DeviceProvisioningService` | Duplicate | Fixed. Now requires a signature\|privileged permission where it had none. |

Eight tickets, closed duplicate or infeasible, fixed in the shipping image. I re-checked all eight in the October builds and the fixes are still there.

Two more are fixed on the QPR3 beta (DP11.261002.005) and not yet on stable:

| Ticket | Component / function | Closure | Verdict |
|---|---|---|---|
| 555981160 | telephony-common.jar `ValueParser.retrieveItemsIconId` | Can't Repro | Fixed on QPR3 beta: a `length < 1` guard added before the array allocation. Still unchanged on the current stable image. |
| 538367541 / 543174762 | telephony-common.jar `WapPushOverSms.decodeWapPdu` | Duplicate | Fixed on QPR3 beta: a bounds check (`decodedDataLength + value <= buffer length`) added before the allocation. Still unchanged on stable. |

## Still unpatched, closed with reasoning that doesn't hold

This is not a payment claim. None of these were fixed, so no reward is owed. It shows the stated closure reason doesn't match the code: the defect is still there, unchanged, in the current factory image. Every row is a closed report.

| Ticket | Component / function | Closure | Verdict |
|---|---|---|---|
| 561846309 | ConfigUpdater.apk `PhenotypeHelper.getCertificate()` | WAI | Unchanged. Returns one certificate for every config type while its sibling methods switch on type. |
| 552305734 | CrossDeviceAccessServicePrimary.apk `PermissionBroadcastReceiver` | Infeasible | Unchanged. Exported, no permission. |
| 558478942 | DCMO.apk `DcmoReceiver` | Duplicate | Unchanged. Exported, no permission. |
| 557568085 | TelephonyProvider.apk `updateSynchronized()` | Duplicate | Unchanged. Caller-supplied selection concatenated with no parentheses before the ownership clause. |
| 555984702 | grilservice.apk `RadioConfig.readFromParcel` | Can't Repro | Unchanged. Raw `readInt()` size used as an allocation, no upper bound. |
| 552188030 | services.jar `Vpn.setVpnForcedLocked()` | Duplicate | Unchanged. UID 0 excluded from the VPN lockdown range. |
| 558472707 | service-npumanager.jar `cancelModelLoad` | Infeasible | Unchanged. No caller-identity check, unlike sibling methods on the same class. |
| 537920968 | nfc_nci.st21nfc.default.so `handlePollingLoopData` | Infeasible | Unchanged. Same undersized length-check logic at the function's new address. |
| 538311587 | android.hardware.secure_element-service.uicc | Infeasible | Unchanged. Same mutex/slot/type offsets, no per-call correlation token. |
| 538314136 | IntentResolver.apk `UriMetadataReaderImpl.getMetadata` | Infeasible | Unchanged. No ownership check on the cross-profile URI read, after a refactor. |
| 561843639 | libmtkmipc-ril.so `handleUssdInd` | (reply only) | Unchanged, recompiled only. Wrong TLV length still used as the copy length. |
| 529102332 | ShannonRcs.apk `DebugBroadcastReceiver` | Duplicate | Unchanged. Exported, no permission. |
| 541986810 | framework.jar `RecoverySystem.verifyPackage` | Duplicate | Unchanged. The certificate checked and the certificate used for verification can still differ. |
| 547481980 | services.jar `FileService.enqueueOperation` | Duplicate | Unchanged (high confidence). No ownership or permission check before dispatch. |

One MediaTek RIL finding, 555425202 (`onPcoInfoNotify`), changed in the QPR3 build. I'm re-verifying it before I call it either way, so it's left out of both tables rather than guessed at.

## Count

- Fixed in shipped code, no reward or credit: eight on stable, two on the QPR3 beta. Ten, each its own ticket.
- Still present, closed with reasoning the code contradicts: the tickets above, plus more Samsung and MediaTek submissions through the same program.
- Could not be determined: a few where a build or device mismatch, a missing partition, or a stripped binary made the diff unreliable. Marked as such, not guessed.

## It isn't only me

In June 2026 The Register covered Justin O'Leary and his "ConfigConfusion" flaw in Google Config Connector, where a user in one Kubernetes namespace could reach GCP organization-owner rights. Google replied "Nice catch!", rated it P1/S1 and accepted, then reversed to "working as intended," denied the bounty, and left it unpatched with the tracker still showing it accepted. Acknowledged, then denied, still unfixed.

CSO Online covered the broader pattern in 2020 (J.M. Porup, *"Bug bounty platforms buy researcher silence, violate labor laws, critics say"*): critics, including legal scholars such as Amit Elazari and Katie Moussouris, argue these programs can be used to suppress disclosure, that researchers often have no legal safe harbor, and that the model raises labor-law questions. The "disposable contractor whose finished work gets rejected with no recourse" framing is mine, not theirs.

## I'm not against Google

One of my own projects, Box, started as a fork of their AI Edge Gallery, and I've worked on it most days for the past year. Google has pulled features from my fork back into their project, so it has gone both ways and in good faith. This is specifically about what happened when I reported real, working security bugs through their own process: they used the work, and most of the time they didn't pay for it.
