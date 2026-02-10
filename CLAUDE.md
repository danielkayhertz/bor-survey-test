# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## Project Overview

This project supports **Impact for Equity's** candidate survey for the **Cook County Board of Review (BoR)** election. The survey collects standardized responses from candidates running for BoR Commissioner seats across Cook County districts.

## Key Files

- `Sample BoR text.docx` -- Template/sample showing the survey structure with placeholder (lorem ipsum) responses
- `impact-for-equity-brand.skill` -- Packaged Claude Code skill for applying IFE brand guidelines

## Document Structure

The survey docx follows a consistent hierarchy:

- **H1 (Heading1)**: District number (e.g., "District 1")
- **H2 (Heading2)**: Candidate label (e.g., "Candidate A")
- **ListParagraph**: Survey question or field label
- **Normal**: Candidate's response text

### Survey Fields Per Candidate

Each candidate section contains these fields (as list items), each followed by a response paragraph:

1. **Name**
2. **Occupation/experience**
3. **Website**
4. **Top 3 priorities** for the Cook County Board of Review and their district
5. **Transparency and fairness** -- increasing transparency, promoting fairness, standardizing appeal procedures
6. **Accessibility and outreach** -- ensuring the appeal process is accessible, communicating guidance, engaging broad audiences
7. **Collaboration with Assessor's Office** -- information sharing with the Cook County Assessor's Office
8. **Systemic inequities** -- addressing shifting property tax burden onto residential homeowners in low-income communities and communities of color; barriers to intergenerational homeownership

## Context

The Cook County Board of Review is a 3-commissioner body that hears property tax assessment appeals. Each commissioner represents a district within Cook County. The survey focuses on property tax equity, appeal process transparency, and community engagement -- core concerns for housing advocates.

## Brand Guidelines (Impact for Equity)

When creating web or print content, apply IFE brand standards:

- **Primary colors**: Navy #003F70, Blue #005198, Sky #2899D5, Light Blue #84B8E3
- **Accent colors**: Orange #F37021 (CTAs only), Lime #CBDC00, Slate #67808E
- **Text**: Charcoal #51585E (never pure black)
- **Typography**: Gotham Bold/Book primary; Calibri Bold/Regular fallback
- **Tone**: Resilient, Methodical, Active, Transparent

## Environment Notes

- Platform: Windows
- Python is available
- DOCX editing toolchain scripts are in the sibling `Advocate's Guide` project at `../Advocate's Guide/scripts/`
