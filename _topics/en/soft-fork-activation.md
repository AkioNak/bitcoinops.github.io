---
title: Soft fork activation

## Optional.  Shorter name to use for reference style links e.g., "foo"
## will allow using the link [topic foo][].  Not case sensitive
# shortname: foo

## Optional.  An entry will be added to the topics index for each alias
aliases:
  - BIP8
  - BIP9

## Required.  At least one category to which this topic belongs.  See
## schema for options
categories:
  - Consensus Enforcement
  - Soft Forks

## Required.  Use Markdown formatting.  Only one paragraph.  No links allowed.
## Should be less than 500 characters
excerpt: >
  **Soft fork activation** describes the moment when a Bitcoin full
  node begins to enforce one or more additional consensus rules.
  These transitions introduce a coordination risk between nodes on the network, so developers have devoted much
  effort over the years to creating and improving soft fork activation
  mechanisms that can minimize the chance a problem will occur.

## Optional.  Use Markdown formatting.  Multiple paragraphs.  Links allowed.
extended_summary: |
  Soft forks allow the overall network to transition to new consensus
  rules even if not every node has adopted those rules.  However,
  anytime different nodes enforce different rules, there's a risk that
  blocks violating the nonuniform rules will be accepted by some nodes
  and rejected by other nodes, causing a consensus failure (chain split)
  that can result in double spends of confirmed transactions and a loss
  of confidence in the safety of the Bitcoin system.  This is the main
  problem that activation mechanisms attempt to mitigate.

  ### History

  Proposals for new soft fork activation mechanisms are often designed
  to avoid problems experienced during previous soft forks, so this
  section attempts to provide an overview of past activation attempts.

  #### [2009] Hardcoded height: consensus nLockTime enforcement

  The [earliest known][earliest sf] soft fork was implemented in Bitcoin
  0.1.6 (released November 2009) and was hardcoded to activate at block
  31,000, which occurred on 22 December 2009.  This *hardcoded height*
  activation mechanism was used for at least one [other][block size sf]
  early soft fork when most development was done by Satoshi Nakamoto.

  #### [2011] Hardcoded time and manual intervention: the BIP12 `OP_EVAL` failure

  After Nakamoto left development, the first soft forking code merged
  into Bitcoin was [BIP12][] `OP_EVAL`.   This planned to use a
  combination of a *hardcoded time* activation method and manual
  intervention if less than 50% of the network hashrate was prepared to
  enforce the change.  Quoting from BIP12:

  > [...] new clients and miners will be coded to interpret OP_EVAL as a
  > no-op until February 1, 2012. Before then, miners will be asked to
  > put the string "OP_EVAL" in blocks that they produce so that hashing
  > power that supports the new opcode can be gauged. If less than 50%
  > of miners accept the change as of January 15, 2012 the rollout will
  > be postponed until more than 50% of hashing power supports OP_EVAL
  > (the rollout will be rejected if it becomes clear that a majority of
  > hashing power will not be achieved).

  The manual intervention may have been needed in this case as a
  [serious vulnerability][oconnor recursion] in `OP_EVAL` was discovered
  after the activation code was merged but before it was released.
  Although that specific bug was [fixed][vanderlaan fix], some
  developers feared that there might be additional problems with such a
  powerful new opcode and the soft fork attempt was abandoned.

  #### [2012] Hardcoded time and manual intervention tried again: BIP16 P2SH

  Several simplified replacements for `OP_EVAL` were then proposed (see
  BIPs [13][BIP13]/[16][BIP16], [17][BIP17], [18][BIP18], and
  [19][BIP19], among [other ideas][maxwell half dozen]), with BIPs 13/16
  Pay-to-Script-Hash (P2SH) [gaining the most traction][p2sh votes]
  among developers.  P2SH [used][7bf8b7c] the same activation mechanism
  as `OP_EVAL`.  The first planned activation was 1 March 2012, but by
  the February 15th polling date, fewer than half of the last 1,000
  blocks had indicated readiness to enforce the BIP16 rules.  A second
  poll of the thousand most recent blocks occurred on March 15th; this
  had sufficient support and so developers released [Bitcoin 0.6.0][] on
  March 30th with an activation date of April 1st.

  #### [2012] Hardcoded time: BIP30 rejecting duplicate txids

  As P2SH activation was being worked on, it was discovered that
  multiple transactions could be created with the same txid.  By itself,
  this bug would only destroy the funds of a user who attempted to
  exploit it, but it could be combined with an oddity in Bitcoin's
  merkle tree construction to break consensus between nodes (see
  [CVE-2012-2459][]).  The first soft fork fix for this was [BIP30][],
  which simply marked as invalid transactions with the same txid as
  another transaction with an unspent output.  The fix was
  non-controversial among the development team and was [activated][bip30
  commit] by a simple hardcoded time included in the same [Bitcoin
  0.6.0][] release that contained P2SH's activation parameters.

  #### [2012] IsSuperMajority (ISM): BIP34 coinbase prefixes

  Despite BIP30 fixing the immediate problem with duplicate txids,
  Bitcoin developers knew they didn't want to have to search an index of
  every previous transaction with unspent outputs each time a new
  transaction was received, so a second solution was developed that
  would eliminate the weakness that made duplicating txids practical.
  This was [BIP34][].  For this change, developers used something
  similar to the miner signaling method from BIP16 P2SH, but this time
  asked miners who were ready for BIP34 to increment their block
  `nVersion` value.  More notably, developers automated enforcement for
  the new rules in the Bitcoin code so they could release soft-fork
  ready software while still waiting for miners to upgrade.  The rules
  from BIP34 were implemented in a function called `IsSuperMajority()`
  (ISM), which initially included a [single activation threshold][ism
  rule 1] that would start enforcement of BIP34's new consensus rule:

  > - 75% rule: If 750 of the last 1,000 blocks are version 2 or
  >   greater, reject invalid version 2 blocks.

  Before release, it was decided to add a [second activation
  threshold][ism rule 2] that would actually fix the underlying problem
  by requiring BIP34 to be used:

  > - 95% rule: If 950 of the last 1,000 blocks are version 2 or
  >   greater, reject all version 1 blocks.

  #### [2015] ISM and validationless mining: the BIP66 strict DER activation

  In September 2014, Pieter Wuille [discovered][strictder disclosure] a
  divergence in the way OpenSSL handled DER-encoded signatures on
  different platforms.  This could be used, for example, to create a
  block that would successfully validate on Linux but fail on
  Windows---making it possible for an attacker to create a targeted
  chain split.  Wuille and a few other developers privately developed the patch for a soft fork that would ensure all signatures
  used the same format.  BIP66 was created for the change and advertised
  publicly as a step towards removing Bitcoin's dependency on OpenSSL
  (which was an actual goal and was finally [achieved][news78 openssl] in 2019).
  After BIP66 gained adequate support from users and
  developers---including from many people who didn't know about the
  security vulnerability---it was released using the same ISM activation
  mechanism as BIP34 used, except using v3 blocks (meaning v2 blocks, or
  earlier, would be rejected at the 95% threshold).

  The 75% threshold was reach at block height 359,753.  The 95%
  threshold was reached at block 363,725 (4 July 2015) and all
  nodes running [Bitcoin Core version 0.10.0][] or later (or a
  compatible implementation) began enforcing the new rules.  However, at
  block height 363,731, one of the miners who had not been signaling
  support for the soft fork produced a version 2 block that was not
  valid under the new ISM activation rules.  It was known this could
  happen up to several times a day until the last few percent of the
  network hash rate upgraded to enforce the new rules, and the expected
  outcome was that upgraded nodes and miners would simply ignore the
  invalid block.  Unfortunately, many miners at the time performed
  *validationless mining* where they would accept a claim that there was
  a new block even before the new block had arrived and been trustlessly
  validated.  When this happened, they would begin mining to extend the
  chain with the invalid block, making any blocks they produced also
  invalid.  This happened after the original block 363,731 was
  produced---miners produced a chain of six invalid blocks before
  developers were able to contact pool operators and get them to
  manually reset their software to return to the valid chain.

  These invalid blocks did not affect users who had upgraded their full
  nodes to enforce the BIP66 soft fork, but users of lightweight clients
  and earlier nodes could've seen the 96 transactions in the original
  block 363,731 receive up to six confirmations even though they weren't
  confirmed in a valid block.  A second event of this type occurred the
  next day, giving transactions three false confirmations.  Eventually,
  enough miners improved their validationless mining software or
  upgraded their nodes and the accidental chain splits stopped occurring.

  In response to the problems of these [July 2015 chain forks][],
  developers redoubled their efforts on reducing the need for
  validationless mining and began working on a new activation mechanism.

  #### [2015] ISM one last time: the BIP65 `OP_CHECKLOCKTIMEVERIFY` activation

  A soft fork to add a new `OP_CHECKLOCKTIMEVERIFY` (CLTV) opcode to
  Bitcoin was proposed prior to the BIP66 strict-DER soft fork but was
  delayed by the urgency of fixing the OpenSSL vulnerability.  This
  illustrated another one of the weaknesses of the ISM mechanism using
  incrementing version numbers---a miner who signals readiness for
  the latest proposal (version *n*) also implicitly signals readiness
  for all previous proposals (e.g. version *n-1*).  This limited the
  ability to use ISM coordinate multiple upgrades in parallel.

  However, despite the problems at the end of BIP66's activation, ISM
  was again used to avoid delaying BIP65 further.  There were no
  activation problems this time.

  #### [2016] BIP9 versionbits: the BIP68/112/113 relative locktime activation

  Several flaws in the ISM mechanism had become apparent by this point
  and so the new [BIP9][] versionbits mechanism was proposed to fix them:

  - **Unnecessary penalizing miners:** when ISM activated and the
    minimum block version was incremented, any miners who failed to
    increment their version would produce invalid blocks even if those
    blocks didn't violate any of the soft fork's other rules.  For
    example, in the block that triggered the 4 July 2015 chain split,
    all the transactions followed the BIP66 strict DER rules---the
    only reason that block and the subsequent five blocks were rejected,
    causing over $50,000 in losses, was that the miner assigned the 
    version number to be 2 instead of 3. 

  - **Impractical parallelization:** with ISM, developers felt they
    needed to wait for one fork to activate before a second fork could
    be started.

  - **Inability to fail:** ISM attempts didn't have expiration dates.
    Once node software was released that waited for the activation
    signal, all nodes from that point needed to continue monitoring for
    that signal until activation occurred.  There was no way to decide
    the soft fork was no longer needed.

  - **Loose measurements and unpredictable activation times:** ISM
    activated whenever 950/1000 blocks signaled readiness.  However,
    because Bitcoin block production is stochastic, a lucky set of
    miners with much less than 95% of the total network hashrate could
    sometimes produce 950 of the last 1,000 blocks, leading to premature
    activation.  Additionally, not knowing the exact activation time in
    advance meant it was difficult for protocol developers, merchant
    system administrators, and mining pool operators to be available in
    the hours shortly after activation occurred in case there were any
    problems that needed a quick response.

  BIP9 versionbits attempted to solve these problems by using the block
  header version field as a bitfield.  Bits in the field were used for
  signaling only---never to reject a block as invalid---and they could
  be set in parallel.  Measurements were taken only once every 2,016
  blocks, minimizing the window during which a small percentage of
  hashrate would need to be lucky to project a false impression of 95%
  signaling.  Finally, when the 95% signaling threshold was reached,
  there would be a 2,016 block (roughly two week) "lock in" period until
  activation occurred to allow people to prepare.  If the signaling
  threshold wasn't reached by a timeout date, the attempt ended,
  allowing the unused code to be removed from later node software
  releases.

  This activation method was first tested with the soft fork that added
  [BIP68][] consensus-enforced sequence numbers, [BIP112][]
  `OP_CHECKSEQUENCEVERIFY`, and [BIP113][] nLockTime enforcement by
  median time past.  The fork quickly made it to the locked-in phase,
  and then proceeded automatically to activation.

  #### [2016-7] BIP9, BIP148, and BIP91: the BIP141/143 segwit activation

  The segwit soft fork was implemented to use the [BIP9][] activation
  method.  A few miners quickly began signaling readiness but support
  peaked far below the needed 95% threshold.  Some Bitcoin users felt
  miners were illegitimately delaying a useful new feature and supported
  a mandatory activation mechanism specified in [BIP148][].  The final
  form of BIP148 specified rejecting any
  blocks that didn't signal support for BIPs 141/143 starting on a
  certain date.

  BIP148 created the risk of a consensus failure.  If miners failed to
  signal BIP9 support for segwit, and most users continued to consider those
  miners' blocks as valid, users of BIP148 might accept bitcoins that
  most other Bitcoin users would consider invalid.  Alternatively, if
  most users of Bitcoin followed the BIP148 rules---but miners still
  produced many BIP148-invalid blocks---those users who didn't enforce
  the BIP148 could accept bitcoins that the BIP148 users would consider
  invalid.  Only if miners were persuaded to follow the BIP148 rules (or
  if no users tried to enforce the BIP148 rules) would safety be
  assured.

  Around the time BIP148 began receiving significant public support,
  several miners, exchanges, and other businesses signed their support
  for a two-step proposal that started with activation of segwit in a
  way that would follow the BIP148 rules.  This first stage was proposed
  in [BIP91][], which was a modification of the BIP9 rules.  Miners used
  a BIP9 versionbit to signal whether they would enforce a temporary
  rule to reject any blocks not signaling the BIP141/143 segwit
  versionbit.  Compared to BIP9, the BIP91 threshold was reduced from
  95% to 80% and its monitoring and lockin period lengths were reduced
  from 2,016 blocks to 336 blocks.

  BIP91 locked in and activated almost as soon as was possible.
  Subsequently, BIP141/143 locked in and activated.  BIP148 mandatory
  signaling expired when BIP141/143 activated.  The second stage of
  the proposal from miners, exchanges, and other businesses required a
  hard fork; spokespersons for the signers eventually withdrew their
  proposal after vocal opposition from a large number of individual
  users and several businesses.

  It remains a subject of debate exactly how much each of the above
  events, and other events occuring around the same time, actually
  contributed to segwit being activated.

  ## Future activations

  After the problems of the segwit activation, several people came to
  work on the [BIP8][] proposal that proponents claims resolves several
  issues with BIP9:

    - **Allows mandatory activation:** BIP8 began as a generalization of
      BIP148, allowing miners to voluntarily signal for activation for
      most periods of the activation but requiring them to signal during
      the final period or risk creating invalid blocks.  Later, a
      parameter `LockinOnTimeout` (LOT) was created to control this
      behavior; nodes using LOT=true will require signaling in the final
      period; nodes using LOT=false will not require signaling but will
      still enforce the new rules if enough blocks are signaling.

    - **Heights instead of times:** BIP9 starts and stops monitoring for
      activation signals based on an average of the times encoded by
      miners into recent blocks.  It's possible for miners to manipulate
      these times, which can stymie a LOT=true activation, so the
      BIP8 proposal instead uses block heights.

  BIP8's flexibility allowed it to be used to compare a [variety of
  activation proposals][] for the proposed [taproot][topic taproot] soft
  fork, although detractors (including BIP8's own champion) have
  leveled several criticisms against aspects of it.  As of this writing,
  discussion in the context of taproot activation remains ongoing.

  Other activation ideas have also been discussed, including
  probabilistic soft fork activation ([sporks][]), multi-stage
  ("modern") soft fork activation ([MSFA][]), decreasing threshold
  activation ([decthresh][]), and a return to hardcoded height or time
  activations ([flag days][]).

  [oconnor recursion]: https://github.com/bitcoin/bitcoin/issues/729
  [vanderlaan fix]: https://github.com/bitcoin/bitcoin/pull/730
  [earliest sf]: https://bitcoin.stackexchange.com/questions/90229/nlocktime-in-bitcoin-core/99104#99104
  [slush /p2sh/]: https://bitcointalk.org/index.php?topic=56969.msg710514#msg710514
  [bitcoin 0.6.0]: https://bitcoin.org/en/release/v0.6.0
  [CVE-2012-2459]: https://bitcointalk.org/?topic=102395
  [block size sf]: https://github.com/bitcoin/bitcoin/commit/8c9479c6bbbc38b897dc97de9d04e4d5a5a36730#diff-608d8de3fba954c50110b6d7386988f27295de845e9d7174e40095ba5efcf1bbR1421-R1423
  [maxwell half dozen]: https://bitcointalk.org/index.php?topic=58579.msg690093#msg690093
  [7bf8b7c]: https://github.com/bitcoin/bitcoin/commit/7bf8b7c25c944110dbe85ef9e4eebd858da34158
  [p2sh votes]: https://en.bitcoin.it/wiki/P2SH_Votes
  [bip30 commit]: https://github.com/bitcoin/bitcoin/commit/a206b0ea12eb4606b93323268fc81a4f1f952531#diff-34d21af3c614ea3cee120df276c9c4ae95053830d7f1d3deaf009a4625409ad2R1271
  [ism rule 1]: https://github.com/bitcoin/bitcoin/commit/de237cbfa4c1aa7d4f9888e650f870a50b77de73#diff-34d21af3c614ea3cee120df276c9c4ae95053830d7f1d3deaf009a4625409ad2R1829-R1841
  [ism rule 2]: https://github.com/bitcoin/bitcoin/commit/d18f2fd9d6927b1a132c5e0094479cf44fc54aeb#diff-34d21af3c614ea3cee120df276c9c4ae95053830d7f1d3deaf009a4625409ad2R1829-R1837
  [strictder disclosure]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2015-July/009697.html
  [news78 openssl]: /en/newsletters/2019/12/28/#openssl
  [july 2015 chain forks]: https://en.bitcoin.it/wiki/July_2015_chain_forks
  [bitcoin core version 0.10.0]: https://bitcoin.org/en/release/v0.10.0#bip-66-strict-der-encoding-for-signatures
  [sporks]: /en/newsletters/2019/02/05/#probabilistic-bitcoin-soft-forks-sporks
  [msfa]: /en/newsletters/2020/01/15/#discussion-of-soft-fork-activation-mechanisms
  [decthresh]: /en/newsletters/2020/07/22/#mailing-list-thread
  [flag days]: /en/newsletters/2021/03/10/#flag-day
  [variety of activation proposals]: https://en.bitcoin.it/wiki/Taproot_activation_proposals

## Optional.  Produces a Markdown link with either "[title][]" or
## "[title](link)"
primary_sources:
    - title: BIP9
    - title: BIP8

## Optional.  Each entry requires "title", "url", and "date".  May also use "feature:
## true" to bold entry
optech_mentions:
  - title: Presentation about probabilistic soft fork activation ("sporks")
    url: /en/newsletters/2019/02/05/#probabilistic-bitcoin-soft-forks-sporks

  - title: Consensus cleanup soft fork proposal (plans for BIP9 activation)
    url: /en/newsletters/2019/03/05/#cleanup-soft-fork-proposal

  - title: Draft taproot proposal deliberately omits activation mechanism
    url: /en/newsletters/2019/05/14/#no-activation-mechanism-specified

  - title: Activation heights for previous soft forks hardcoded into Bitcoin Core
    url: /en/newsletters/2019/08/21/#hardcoded-previous-soft-fork-activation-blocks

  - title: Mailing list discussion of soft fork activation methods
    url: /en/newsletters/2020/01/15/#discussion-of-soft-fork-activation-mechanisms

  - title: "`OP_CHECKTEMPLATEVERIFY workshop discussion about activation mechanisms"
    url: /en/newsletters/2020/02/12/#op-checktemplateverify-ctv-workshop

  - title: Meetup discussion including potential soft fork activation mechanisms
    url: /en/newsletters/2020/06/03/#sydney-meetup-discussion

  - title: "BIPs #550 revises BIP8 to allow use without initial mandatory lock-in"
    url: /en/newsletters/2020/07/01/#bips-550

  - title: Taproot activation discussion on mailing list and in new IRC room
    url: /en/newsletters/2020/07/22/#taproot-activation-discussions

  - title: Conversation about segwit activation history and ideas for taproot
    url: /en/newsletters/2020/09/02/#taproot-activation

  - title: "Changes 'unexpected version' warnings implemented as part of BIP8"
    url: /en/newsletters/2020/10/07/#bitcoin-core-19898

  - title: "Bitcoin Core #19953 adds taproot code; activation still under discussion"
    url: /en/newsletters/2020/10/21/#bitcoin-core-19953

  - title: Survey released with developer preferences for soft fork activations
    url: /en/newsletters/2020/11/04/#taproot-activation-survey-results

  - title: Website listing miner support for taproot activation
    url: /en/newsletters/2020/11/25/#website-listing-miner-support-for-taproot-activation

  - title: Q&A comparison of different soft fork activation mechanisms
    url: /en/newsletters/2020/12/16/#how-do-bip8-and-bip9-differ-how-are-they-alike

  - title: "2020 year-in-review: soft fork activation mechanisms"
    url: /en/newsletters/2020/12/23/#activation-mechanisms

  - title: Meeting scheduled to discuss taproot activation
    url: /en/newsletters/2021/01/27/#scheduled-meeting-to-discuss-taproot-activation

  - title: Taproot activation meeting summary and follow-up meeting schedule
    url: /en/newsletters/2021/02/10/#taproot-activation-meeting-summary-and-follow-up

  - title: "BIPs #1021 eases BIP8 LockinOnTimeout=True signaling requirements"
    url: /en/newsletters/2021/02/10/#bips-1021

  - title: "BIPs #1021 updates BIP8 to no longer require signaling during lockin period"
    url: /en/newsletters/2021/02/10/#bips-1020

  - title: "Summary of second taproot activation meeting and mailing list discussion"
    url: /en/newsletters/2021/02/24/#taproot-activation-discussion

  - title: "BIPs #1069 makes BIP8 signal threshold configurable and reduces default"
    url: /en/newsletters/2021/03/03/#bips-1069

  - title: "Taproot activation mailing list discussion; Speedy Trial attempt proposed"
    url: /en/newsletters/2021/03/10/#taproot-activation-discussion

## Optional.  Same format as "primary_sources" above
see_also:
  - title: BIP activation heights, times, and thresholds
    link: https://gist.github.com/ajtowns/1c5e3b8bdead01124c04c45f01c817bc
  - title: Taproot
    link: topic taproot
---