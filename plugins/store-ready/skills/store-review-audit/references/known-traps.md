# Known traps (field-tested)

These come from real near misses and upload errors on shipped apps. Check
every one that applies, and add new ones you find to the report's hygiene
section so the project can learn from them.

1. A disabled config flag does not remove a framework from the binary.
   Purpose-string errors come from SDKs you forgot you had. Scan the binary.
2. A submission that contains only the version, while the IAPs or
   subscriptions sit in "Ready to Submit", reaches the reviewer with a broken
   store. Check the item count.
3. The uploaded build can be older than the last fixes. Compare build time
   with the last commit.
4. A new version may inherit the previous screenshots; append-style uploads
   then exceed or duplicate the set. Check by hash.
5. Promo text and What's New can be empty in every locale after a version is
   created through the API. Diff every localized field against intent.
6. A marketing or support page that says "work in progress", or advertises
   features the build does not have, is metadata the reviewer reads.
7. Terms, trial lengths and prices written in the Terms page, paywall and
   notes drift apart. They must match the store configuration.
8. A demo account can show an active subscription from old sandbox
   purchases, so the reviewer cannot test buying.
9. An icon exported as RGBA fails even when every pixel looks opaque.
10. `supportsTablet`-style flags can add iPad orientations you did not mean
    to support.
11. Pre-permission screens with an "Allow" button, permission prompts
    stacked at launch, and prompts over gameplay draw rejections and bad
    reviews.
12. Promised event rewards, badges or content missing from the build make
    the listing inaccurate. Check store promises against the build, not only
    the console form.
13. A new version cannot be created while another is waiting for review.
    Plan metadata changes accordingly.
14. Missing EU DSA trader status silently removes the app from EU
    storefronts. A trader form can be rejected for details that differ from
    the uploaded document (address format, region fields).
15. Review traffic pollutes analytics: Apple's sample attribution payload
    (campaign id `1234567890`) and Google's automated test devices show up
    after each release. Tell the user to exclude them.
16. Store-console answers written from a template (Play production access,
    age rating) can claim things that did not happen. Read them back.
17. Unverifiable numbers in screenshots ("loved by millions") are a
    metadata finding even when they feel like marketing.
18. Words banned by the project's own copy rules slip into one locale.
    Check all of them.
19. A resubmission is a new review. An app rejected for one issue (an iPad
    layout bug, a missing IAP) can come back with a 4.3 or a 2.1(b) it did
    not get the first time. After a fix, rerun the whole audit, not only the
    fixed item.
20. A feature removed from the app stays on the paywall. A pre-submission
    audit caught a paywall still selling a checklist that had been cut two
    days earlier. Diff the paywall and IAP copy against the build, starting
    with what was removed since the live version.
