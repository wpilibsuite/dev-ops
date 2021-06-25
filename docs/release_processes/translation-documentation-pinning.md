# Documentation & Translation Pinning

This document details the process of pinning and versioning the [translation project](https://github.com/wpilibsuite/frc-docs-translations) for the FIRST/WPILib documentation. Pinning the versions of translations is very important for a couple reasons.

- Stability
- Consistency
- Ease of Translation

Translations are accessible via the language code. IE: [https://docs.wpilib.org/es/latest/](https://docs.wpilib.org/es/latest/).

## Our Approach

Translations happen consistently throughout the year, as long as there is documentation momentum. To make it easy to understand, we break this process up into two parts.

Reference: ``/latest/`` is the "leading edge" branch of the docs. Accessible from [https://docs.wpilib.org/en/latest/](https://docs.wpilib.org/en/latest/) and ``/stable/`` is the "year-to-year" version of the docs. This is accessible from [https://docs.wpilib.org/en/stable/](https://docs.wpilib.org/en/stable/)

Beginning of Season:
- Translation and Documentation ``/latest/`` and ``/stable`` are at an equal point in history. This allows us to make "bleeding" edge changes to documentation and translations. The benefit of this, is making it immediately live on both versions of the site.

End of Season:
- Translations & Documentation have ``stable`` locked for new **content** additions but are able to be translated. ``latest`` is unable to be translated at this time. 
- ``latest`` for the [documentation repo](https://github.com/wpilibsuite/frc-docs) is tagged at a yearly date for archival purposes (IE: 2022).

Mid-Summer:
- Translations have ``stable`` locked for new **content** and new **translations**. ``latest`` is now available for translation. Translation repo is tagged at a yearly date (IE: 2022).

This release cycle ensures that our visitors always get the most relevant and stable documentation. We don't want visitors receiving development changes to documentation for things they are still working on for the current year.
