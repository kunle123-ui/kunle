| Release (tag)                 | Date        | Version     | Comments             |
|-------------------------------|-------------|-------------|----------------------|
| Dawn v1.0.0 (dawn-v1.0.0)     | 2017 Sep 14 | 6294dea [1] | First beta release   |
| Dawn v2.0.0 (dawn-v2.0.0)     | 2017 Dec 4  | 9703495     | Second beta release  |
| STAT-2017-12-12            | 2017 Dec 12 | 00db493  | reverts primary genesis.json|
| STAT-2017-12-13            | 2017 Dec 13 | 941fa2f  | connect via remote **`nodeos`** |
| STAT-2017-12-21            | 2017 Dec 21 | 233ba4e  | p2p stability and other fixes |
| DAWN-2018-01-05            | 2018 Jan 5 | 2cc40a4  | Stability release for DAWN 2.x |
| DAWN-2018-01-09-ALPHA            | 2018 Jan 9 | ede87e6  | Stability release for DAWN 2.x |
| DAWN-2018-01-18-ALPHA            | 2018 Jan 18 | 466b4ce  | Stability release for DAWN 2.x |
| DAWN-2018-01-23-ALPHA            | 2018 Jan 23 | 26eecaa  | local nodeos network updates for DAWN 2.x |
| DAWN-2018-01-31-ALPHA            | 2018 Jan 23 | d75c30e  | DAWN 3.0 pre-release, single node only |
| DAWN-2018-02-12           | 2018 Feb 12 | 6992cf4  | Stability release for DAWN 2.x |
| DAWN-2018-02-14           | 2018 Feb 14 | bcb5bf7  | Stability release for DAWN 2.x |
| SUPERDAWN-2018-03-02           | 2018 Mar 2 | 8a9f412  | Pre-release for DAWN 3.0 |
| SUPERDAWN-2018-03-04           | 2018 Mar 4 | 4274b19  | Pre-release for DAWN 3.0 |
| SUPERDAWN-2018-03-18           | 2018 Mar 8 | a731471  | Pre-release for DAWN 3.0 |
| DAWN-2018-03-23           | 2018 Mar 23 | a7f2601  | Pre-release for DAWN 3.0 |
| DAWN-2018-03-28-ALPHA           | 2018 Mar 28 | 5a18389  | ALPHA Pre-release for DAWN 3.0 |
| DAWN-2018-03-29-ALPHA           | 2018 Mar 29 | d6c3633  | ALPHA Pre-release for DAWN 3.0 |
| DAWN-2018-03-30-ALPHA           | 2018 Mar 30 | 1c1e93d  | ALPHA Pre-release for DAWN 3.0 |
| EOSIO DAWN 3.0           | 2018 Apr 5 | d9ad8ee  | DAWN 3.0 Official Release |

[1] This release is not self-identifying

NOTES: 
* Software releases can be found [here](https://github.com/EOSIO/eos/releases "EOS.IO Releases").
* Tags (bundles of code that go together for deployment, mostly between releases) can be found [here](https://github.com/EOSIO/eos/tags "EOS.IO Software Tags").
* Tags can never provide a document that includes themselves. Therefore the above list, when located in Readme.md, cannot include the latest tag. This same list when found in a wiki can include the latest tag.

[a] #hotfix Beginning in 2018, Hotfixes are tagged as "DAWN" and the year, month, and day of their release, formatted as DAWN-yyyy-mm-dd. If two hotfixes are on one day, the second has a lowercase 'a' appended, the third has a lowercase 'b' appended, etc., as in "DAWN-2018-03-03b". The Hotfix tags can be used by sophisticated developers to download the latest code to come out of each weekly sprint, and to access hotfixes that may be released mid-sprint. There may not be any Release Notes explaining a particular Hotfix release, or such notes may be terse. Documentation is not released on a Hotfix schedule; the public should not expect to find up-to-date documentation connected to any Hotfix tag. (Exception: Doxygen automated documentation will still be up to date with each tag.) . Any tag that is not an official release will also include the suffix "APLHA" to denote that this release may have unresolved bugs.

[b] #release Beginning in 2018, Releases are tagged as "DAWN" and "-R" for Release, then the Major Release number and the Feature Release number, formatted as DAWN-Rmaj.fea. For example, "DAWN-R3.1" – this is Major Release 3, Feature Release 1 within that major release. The Release tags should be used by average developers to download stable releases that have up to date documentation and Release Notes.
