# AI Statement

Version 1.0.0, status is active.

Canonical location is `AI_STATEMENT.md` at the repository root.

Review at least at every major release.

Abstract This document discloses how artificial-intelligence tools are used
to develop this software. It states what the tools do and do not touch, who is
accountable, which controls bound the work and how each is enforced, the
licensing and data posture, the rules for contributors, the uses that are
prohibited, and the limitations that survive all of it. It is a self-declaration
by the maintainer, written for evaluators and regulated adopters performing
supplier due diligence, and it changes in the same pull request that changes the
practice it describes.

The key words shall, should, and may, are used as ISO/IEC Directives Part 2
defines them: requirement, recommendation, permission.

## 1. Scope

This document covers the use of AI tools in developing everything in this
repository.

It does not cover an AI system in the product, because there is none. AI is used
to build the software, in the same sense that autocomplete, syntax linters, and
compilers, are used to build it.

## 2. Which frameworks apply here, and which do not

- The EU AI Act imposes no obligation on this project. The Act binds
  providers and deployers of AI systems (Articles 2 and 3(1)); this software
  is not one. Article 50's marking duties bind the AI tool's provider, not
  the tool's user, and the European Commission's Article 50 FAQ places
  source code outside the content-marking obligation. This document is
  voluntary.
- Medical-device regulation does not classify this project as a device.
  Under MDCG 2019-11 Rev. 1, electronic patient record systems that store,
  transfer, and retrieve records are not qualified as medical devices. A
  downstream integrator who gives their product a medical purpose may bring
  their product into scope; that classification is theirs to make, and
  this document exists partly so they can answer their own supplier
  questions.
- ISO/IEC 42001 and the NIST AI RMF are used as vocabulary, not claimed
  as conformity. No certification is claimed, no audit has occurred, and
  the words "certified," "audited," and "validated" appear in this document
  only inside this sentence, to say they do not apply.

## 3. Terms

This document uses the W3C AI Content Disclosure vocabulary:

- none: entirely human-authored
- ai-assisted: means human-authored; AI edited, refined, or filled in boilerplate
- ai-generated: AI-generated with human prompting and review
- autonomous: AI-generated without meaningful human oversight
- agentic tool: a tool that and executes multi-step work, under a human's direction, as opposed to inline completion.

## 4. Accountability

[MAINTAINERS.md](MAINTAINERS.md) lists the accountable maintainers for
every change in this repository, whatever tool produced the bytes.

## 5. Where AI is used, and at what level

The tooling is agentic AI coding assistance (currently Claude Code, by
Anthropic), operated in sessions the maintainer directs, reviews, and
merges. Levels below use the §3 vocabulary, per development activity.
Deliberately, no percentage appears anywhere in this document: no
defensible method exists for measuring one.

| Activity                                                      | Level        | Notes                                                                                                                                  |
| ------------------------------------------------------------- | ------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| Application and tooling code                                  | ai-generated | written in directed sessions; reviewed and merged by the maintainer                                                                    |
| Tests and the conformance catalogue                           | ai-generated | held to the same authority as the code they test: expectations cite the specification, and the attribution rules below govern failures |
| Documentation and this statement                              | ai-generated | held to the repository's own prose rules                                                                                               |
| Specification adjudications, owner rulings, release decisions | none         | decided by the maintainer and recorded in the spec                                                                                     |
| Contribution and review verdicts on others' work              | none         | not in yes                                                                                                                             |

## 6. Human oversight

The maintainer directs the work. AI is enabled to merge change.

Where the tools run sessions, the session's significantion decisions with
consequences — what a specification silence means, what ships in a release — are
directed to be recorded in the specifications. A decision that exists only
inside a tool session is not a decision this project made.

## 7. Quality controls

- Specification authority. The specifications are vendored
  in the repo, and are the only conformance oracle; specification-facing
  decisions cite file and section. Enforced by the conformance pipeline below
  and by review against the cited text.
- Executed conformance. A committed runner executes a machine-readable
  catalogue of automatic tests cases; the
  published verdicts and reports derive from committed run records, and
  drift gates fail when a regenerated document disagrees with a committed
  one. This is the control that helps catch a plausible-but-wrong
  implementation regardless of who wrote it.
- Static and supply-chain gates. No `unsafe` (compiler-`forbid`),
  deny-tier lints on panicking shortcuts, typed errors, machine-checked
  comment style, `cargo deny` policy.

## 8. Licensing and provenance of AI output

The project is licensed. The position taken here follows the Apache
Software Foundation's and LLVM's published reasoning: an AI tool's output does not launder anyone's copyright, the
full provenance of generated text is generally not knowable, and prompting
alone is not treated as authorship. In practice: contributions of
substantially copied third-party material are refused however they were
produced; generated code is held to the same originality expectations as
human code, under the same review; and if identifiable third-party
material is found in the tree, it is removed or licensed properly, exactly
as it would be for a human-introduced copy. The tools are used under terms
that do not restrict the output's use in licensed software.

## 9. Data

No patient data, no personally identifiable health information, and no
customer data exists anywhere in this project — not in the repository, not
in test fixtures, not in telemetry, and therefore not in any prompt. Test
data is synthetic or comes from the published openEHR corpora and CKM
clinical models, which are modelling artifacts, not records about people.
This is a structural property a reader can check against the tree, not a
promise about tool behaviour. Vendor-side data handling is governed by the
tool vendor's terms; this document deliberately makes no claim on the
vendor's behalf, because such claims go stale silently. The served
software's own telemetry excludes clinical payload content by reviewed
design.

## 10. Rules for contributors

Contributors **may** use AI tools. A contribution with **ai-generated**
content per §3 **shall** say so in the pull-request description — which
tool, and what it did. Disclosure lives in the PR description and never in
commit trailers, for two reasons stated openly: this repository's standing
rule keeps tool attribution out of commit metadata (one maintained
disclosure beats ten thousand trailer lines, and this document is that
disclosure), and the wider ecosystem has no agreed trailer anyway — the
same trailers some communities recommend, others forbid. The contributor
remains responsible for their submission in full, under the same
[CONTRIBUTING.md](CONTRIBUTING.md) bar as any other work: understood,
explained on request, tested, and honest.

## 11. Prohibited uses

In this project, AI **shall not**: merge anything; adjudicate, score, or
answer reviews of contributions (the PR reviewer of §7 is advisory input
to the maintainer, not a verdict); sign anything; decide a
specification-facing question (silences are adjudicated by the maintainer
and recorded); or weaken a test, an expectation, or a gate to make
something pass — the last being a standing hard rule for humans and tools
alike.

## 12. Limitations and residual risks

This section exists because a disclosure without one is marketing.

- **The gates prove what they test, not correctness.** The conformance
  suite demonstrates the behaviours its catalogue covers; coverage is
  broad, ratchets upward, and is itself reviewed — and it is still a
  boundary, published with the run records.
- **Review depth is one person's.** The project has a single maintainer;
  machine gates stand in for the review capacity a larger team would have
  ([GOVERNANCE.md](GOVERNANCE.md) says this plainly). "The maintainer
  understands and can explain every merged change" is the honest claim;
  "every line was independently re-derived" would not be.
- **Retroactivity.** Commits predating this statement carry no disclosure
  markers; this document describes the practice, not a per-commit audit
  trail, and no such trail is claimed.
- **Provenance uncertainty survives.** Whether any generated fragment
  echoes unlicensed training material is not fully knowable with current
  tools; §8 states the handling, not a guarantee.
- **The legal ground is unsettled.** Copyright in AI output is an open
  question in most jurisdictions; this document records positions, and
  positions may have to change. §13 names the triggers.
- **This is a self-declaration.** No third party has audited it. The
  checkable artifacts in §7 are the counterweight: they can disagree with
  this document, and if they do, the document is wrong.

## 13. Review and change

This statement is reviewed at every major or minor release, and revised
off-cycle when any of these fires: the tooling changes materially, a tool
vendor's terms change in a way §8 or §9 relies on, a binding rule emerges
(EU AI Act guidance touching this use, a foundation policy this project
follows, a court decision on AI output and copyright), or a claim in this
document stops being true. The maintainer owns the review; the change
lands as a pull request like everything else, and the version and change
log update in the same PR.

## 14. Reporting

A suspected provenance, licensing, or quality problem in this repository —
including a claim in this document that does not survive checking — is a
report this project wants. Open an issue and cite this file; for anything
security-sensitive, use the private route in [SECURITY.md](SECURITY.md).
The handling commitment is the same as for any defect: attributed,
answered on the tracker, and never silently absorbed.

## 15. References

**Normative for this project** (the documents that bind the practice
described here): the [MIT licence](LICENSE); the vendored openEHR
specifications (`docs/specs/openehr/`); the repository's rule set
(`.claude/rules/`, in particular `reliability.md`, `cnf-triage.md`,
`ai-code-review.md`, `writing-style.md`); [GOVERNANCE.md](GOVERNANCE.md),
[MAINTAINERS.md](MAINTAINERS.md), [CONTRIBUTING.md](CONTRIBUTING.md),
[SECURITY.md](SECURITY.md).

**Informative** (the sources this document's structure and positions draw
on, surveyed 2026-08-24 with the full record on tracker issue #2625):
the W3C AI Content Disclosure vocabulary; the ISO/IEC Directives Part 2
document conventions and verbal forms; RFC 7322's required-considerations
discipline; ICMJE's AI-authorship position; the Apache Software
Foundation's generative-tooling guidance; the Linux Foundation's
generative-AI policy; the Fedora Council's AI-assisted-contributions
policy; the Linux kernel, LLVM, Kubernetes, NumPy, Mozilla, QEMU, curl,
Gentoo, OpenInfra, Ghostty, Kyverno, and nf-core positions; the OpenSSF
security guidance for AI code assistants and the OpenSSF/CNCF maintainer
guide; NIST AI RMF and ISO/IEC 42001 as vocabulary; EU AI Act Articles 2,
3, and 50 with the European Commission's Article 50 FAQ; MDCG 2019-11
Rev. 1.

## Annex A. Change log

| Version | Date       | Change       |
| ------- | ---------- | ------------ |
| 1.0.0   | 2026-08-24 | First issue. |

## Annex B. Machine-readable summary

Levels per the W3C AI Content Disclosure vocabulary (§3); the prose above
is authoritative where the two could ever disagree.

```yaml
ai-statement:
  version: 1.0.0
  last-updated: 2026-08-24
  vocabulary: w3c-ai-content-disclosure
  disclosure-default: ai-generated
  tools:
    - name: Claude Code
      provider: Anthropic
  processes:
    design: ai-assisted
    implementation: ai-generated
    testing: ai-generated
    documentation: ai-generated
    review: none
    adjudication: none
    release-decisions: none
  ships-ai-system: false
  autonomous-use: none
```
