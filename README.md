<!-- README.md -->
<!-- SPDX-FileCopyrightText: Copyright (C) 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors -->
<!-- SPDX-License-Identifier: Apache-2.0 -->
 <picture>
  <img src="assets/images/citecraft_logo_64.png" alt="CiteCraft Logo" align="left" width="64" height="64" style="margin-right: 15px; margin-bottom: 10px;">
</picture>

# CiteCraft-history

**Archived LLM engineering transcripts, design discussions, and implementation histories for the CiteCraft engine.**

---

> [!NOTE]
> **Looking for the code?** This repository contains only historical design documentation and raw LLM conversation transcripts. To view, install, or contribute to the functional codebase, visit the primary repository: **[github.com/sebastien-lenard/citecraft](https://github.com/sebastien-lenard/citecraft)**.

---

## What is CiteCraft?

**CiteCraft** is a lightweight Python utility and command-line interface (CLI) engineered to streamline scientific publishing workflows. It automatically parses `.docx` manuscripts, extracts in-text citations, cross-references them against open scholarly databases, and formats a complete, journal-compliant bibliography.

## Purpose of this Repository

Developing complex data processing workflows via LLMs generates massive conversational transcripts. Storing these heavy text files directly within a core codebase forces regular users to download megabytes of raw history every time they clone or update the project.

To preserve an optimal, lightweight footprint for mobile users and standard developers, all foundational logs are decoupled here. This repository archives:
* Deep-dive architectural and design discussions.
* Step-by-step implementation logs and logic breakdowns.
* Lengthy debugging transcripts and feature-expansion brainstorming sessions that exceeded standard GitHub Issue limits.

## Project Structure

All transcripts are organized chronologically within the `docs/history/` directory for historical baseline tracking and context auditing.