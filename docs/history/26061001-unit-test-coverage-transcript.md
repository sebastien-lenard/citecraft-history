---
status: Finalized Baseline
classification: Immutable Historical Context
date: 2026-06-10
---
<!-- docs/history/26061001-unit-test-coverage-transcript.md -->
<!-- SPDX-FileCopyrightText: Copyright (C) 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# 26061001: Unit Test Coverage Transcript.

---

# ME 
I am going to build a solid context for my project suite. Do not refactor these configuration files; simply read, analyze, and keep them in your active memory to evaluate and implement the unit tests I will request.

[pyproject.toml, utils/config.py, logging_config.py, conftest.py]
```
# pyproject.toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/citecraft"]

[project]
name = "citecraft"
version = "0.1.0"
license = "Apache-2.0"
license-files = ["LICENSE"]
description = "Generates bibliography of manuscripts"
readme = "README.md"
requires-python = ">=3.12.9"
authors = [
    { name = "Sebastien Lenard", email = "sebastien.lenard@gmail.com" }
]
dependencies = [
    "citeproc-py>=0.9.3",
    "click>=8.3.3",
    "customtkinter>=5.2.2",
    "defusedxml",
    "email-validator>=2.3.0",
    "httpx>=0.28.1",
    "idna>=3.15", # addresses vulnerabilities of 3.13
    "pydantic-settings>=2.14.1",
    "python-docx>=1.2.0",
    "python-dotenv>=1.2.2",
    "python-json-logger>=4.1.0",
    "tenacity>=9.1.4",
    "unidecode>=1.4.0",
]

[dependency-groups]
dev = [
    "pytest>=8.3.4",
    "pytest-recording>=0.13.2",
    "ruff>=0.9.0",
]

[project.scripts]
citecraft = "citecraft.cli:cli"

[tool.pytest.ini_options]
minversion = "8.0"
addopts = "-ra -q --tb=short"
testpaths = ["tests"]
pythonpath = ["src"]
markers = [
    "e2e: end to end tests (slower)",
    "integration: integration and api tests (slower)",
    "unit: unit tests (faster)"
]
filterwarnings = [
    "ignore:'citecraft.cli' found in sys.modules:RuntimeWarning"
]

[tool.ruff]
line-length = 88
target-version = "py312"
extend-exclude = [
    "uv.lock"
]
unsafe-fixes = false

[tool.ruff.lint]
select = ["E", "F", "I", "B", "A", "ANN", "ARG", "BLE", "C4", "C90", "COM", "D", "DTZ", "EM", "ERA", "EXE", "FA", "FBT", "FIX", "FLY", "FURB", "G", "ICN", "INP", "ISC", "LOG", "N", "PERF", "PGH", "PIE", "PL", "PT", "PTH", "PYI", "Q", "RET", "RSE", "RUF", "S", "SIM", "SLF", "UP", "T10", "T20", "TC", "TD", "TID", "TRY", "W", "YTT"]
extend-ignore = ["D107"]
fixable = ["ALL"]
unfixable = []
allowed-confusables = ["ℹ"]

[tool.ruff.lint.per-file-ignores]
"tests/**/*.py" = [
    "ARG001", # Allow apparently non accessed arguments
    "ARG005", # Allow apparently non accessed arguments
    "BLE001",  # Allow broad exception catching (great for checking fallback paths)
    "FBT001", # Allow boolean positional arguments
    "FBT002", # Allow boolean default positional arguments
    "PLR2004", # Allow numerical values not being constant
    "PLW0120", # Allow loose control structures if an LLM generated messy loop edge-case tests
    "PLR0913", # Allow many arguments
    "S101",    # Allow 'assert' statements (otherwise bandit flags every single test)
    "SLF001", # Access to private member
]
"__init__.py" = ["D104"]
"__main__.py" = ["D100", "D104"]
# citeproc-py doesnt expose internal exceptions
"src/citecraft/adapters/citeproc_adapter.py" = ["BLE001"]
"src/citecraft/storage/db_client.py" = ["S608"]
"src/citecraft/logging_config.py" = ["T20"]
"tests/e2e/test_cli_pipeline.py" = ["S603"]


[tool.ruff.format]
quote-style = "double"

# src/citecraft/utils/config.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Citecraft environment configuration."""

from functools import lru_cache
from pathlib import Path
from typing import Any

from pydantic import EmailStr, Field, computed_field
from pydantic_settings import BaseSettings, SettingsConfigDict

from citecraft.schemas import (
    HttpsUrlStr,
    UrlWithObjectName,
)


class AppConfig(BaseSettings):
    """Configuration loader and validator for the environment variables."""

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        extra="ignore",
        frozen=True,
        env_nested_delimiter="__",
    )

    # -------------------------------------------- #
    # USER'S CONFIGURATION                         #
    # -------------------------------------------- #

    # --- User's Directory Paths ---
    local_repo_dir_path: Path = Field(default=Path("repo"), alias="LOCAL_REPO_DIR_PATH")
    output_dir_path: Path = Field(default=Path("output"), alias="OUTPUT_DIR_PATH")
    # Warning: log_dir_path is handled in logging_config.py.

    # --- User's API Polite Pool email ---
    user_email: EmailStr

    # --- User's OpenAlex API Key ---
    openalex_api_key: str

    # -------------------------------------------- #
    # OTHER VARIABLES                              #
    # -------------------------------------------- #

    # --- Filepaths ---
    db_filename: str = Field(default="cache.db", alias="DB_FILENAME")

    # --- Default API calls ---

    default_api_key: str | None = None
    default_api_delay: float = 0.5
    default_api_timeout: float = 20
    default_api_max_retry: int = 10
    default_api_url_max_character_length: int = 2048

    # --- Crossref API ---
    crossref_api_journals_url: HttpsUrlStr
    # Contains oject_name placeholder for issn
    crossref_api_journals_issn_url: UrlWithObjectName
    crossref_api_styles_url: HttpsUrlStr
    crossref_api_works_url: HttpsUrlStr
    crossref_api_works_get_limit: int = 20

    crossref_api_key: str | None = None
    crossref_api_delay: float = default_api_delay
    crossref_api_timeout: float = default_api_timeout
    crossref_api_max_retry: int = default_api_max_retry
    crossref_api_url_max_character_length: int = default_api_url_max_character_length

    # --- DOI Service ---
    doi_api_url: UrlWithObjectName  # Contains oject_name placeholder for doi

    # Default API configuration used for DOI Service

    # --- OpenAlex API ---
    openalex_api_works_url: HttpsUrlStr
    openalex_api_works_get_limit: int = 100

    openalex_api_delay: float = default_api_delay
    openalex_api_timeout: float = default_api_timeout
    openalex_api_max_retry: int = default_api_max_retry
    openalex_api_url_max_character_length: int = default_api_url_max_character_length
    openalex_api_url_max_character_length_for_issns_filter: int = 50
    openalex_api_max_piped_filters: int = 100

    # --- Style Repositories ---
    style_repo_url: UrlWithObjectName  # Contains oject_name placeholder for style
    child_style_repo_url: UrlWithObjectName  # Contains template style
    all_styles_repo_url: HttpsUrlStr
    csl_xml_namespaces: dict[str, str]
    # Warning: csl_xml_namespaces with no HttpUrl type that Pydantic could alterate

    # --- Core Logic Settings ---
    min_publication_year: int = 1600
    max_publication_year: int = 2099
    max_count_of_authors_in_citation: int = 2
    context_keywords: str = ""
    journal_update_days: int = 30
    journal_update_limit: int = 100
    default_reference_style: str = "apa"
    default_logging_frequency_for_batch_updates: float = 10.0

    # --- Values for schemas ---
    work_csl_schema_types: list[str] = Field(default_factory=list)

    # --- Blacklists & Cleaners ---
    parser_blacklist: list[str] = Field(default_factory=list)
    work_crossref_schema_blacklist_fields: list[str] = Field(default_factory=list)
    author_crossref_schema_blacklist_fields: list[str] = Field(default_factory=list)
    work_openalex_schema_blacklist_fields: list[str] = Field(default_factory=list)
    author_openalex_schema_blacklist_fields: list[str] = Field(default_factory=list)

    # HTML Cleaning Configuration
    # Structural tags to explicitly preserve in the local repository
    preserved_html_tags: set[str] = Field(default_factory=set)
    # Styling tags whose inner text is kept but boundaries are discarded
    discarded_html_tags: set[str] = Field(default_factory=set)

    @computed_field
    @property
    def db_filepath(self) -> Path:
        """Dynamically construct the db filepath once repo path is resolved."""
        return self.local_repo_dir_path / self.db_filename

    # --- Directory Lifecycle Methods ---
    def ensure_repo_directory(self) -> None:
        """Create the validated local repository directory."""
        self.local_repo_dir_path.resolve().mkdir(parents=True, exist_ok=True)

    def ensure_output_directory(self) -> None:
        """Create the validated output directory."""
        self.output_dir_path.resolve().mkdir(parents=True, exist_ok=True)


def create_config(**kwargs: Any) -> AppConfig:  # noqa: ANN401
    """Instantiate a fresh configuration payload for runtime or isolation testing."""
    return AppConfig(**kwargs)


@lru_cache(maxsize=1)
def get_config() -> AppConfig:
    """Retrieve the globally cached configuration configuration.

    WARNING: Should either be called in cli.py or inside class methods, not outside, so
    as to make tests not interfering with production directories.
    """
    return create_config()

# src/citecraft/logging_config.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""System logging configuration management, formatting, and routing workflows."""

import logging
import logging.config
import os
import sys
import traceback
import uuid
from pathlib import Path
from types import MappingProxyType
from typing import Any

from .utils import get_safe_dir

# Unique session identification tag
RUN_ID: str = str(uuid.uuid4())[:8]


class RunIdFilter(logging.Filter):
    """Filter that automatically injects a unique run_id into logging records."""

    def filter(self, record: logging.LogRecord) -> bool:
        """Inject the static application run identifier into the log record context."""
        record.run_id = RUN_ID
        return True


class ColorFormatter(logging.Formatter):
    """Custom formatter to inject ANSI escape sequences into console logs."""

    GREY = "\033[90m"
    WHITE = "\033[37m"
    YELLOW = "\033[33m"
    RED = "\033[31m"
    BOLD_RED = "\033[31;1m"
    RESET = "\033[0m"

    # Push cursor to start of line and clear current line on terminal
    CLEAR_LINE = "\r\033[K"

    LEVEL_COLORS = MappingProxyType(
        {
            logging.DEBUG: GREY,
            logging.INFO: WHITE,
            logging.WARNING: YELLOW,
            logging.ERROR: RED,
            logging.CRITICAL: BOLD_RED,
        },
    )  # immutable wrapper

    def format(self, record: logging.LogRecord) -> str:
        """Clear current terminal progress line and apply color to the output."""
        orig_name = record.name
        record.name = orig_name.rpartition(".")[-1]
        try:
            result = super().format(record)
        finally:
            record.name = orig_name  # Restore to avoid breaking other handlers

        color = self.LEVEL_COLORS.get(record.levelno, self.RESET)
        return f"{self.CLEAR_LINE}{color}{result}{self.RESET}"


def get_logging_config(log_dir: Path, verbose_level: int = 0) -> dict[str, Any]:
    """Build the configuration dictionary for system loggers."""
    match verbose_level:
        case 1:
            console_level = "INFO"
        case lvl if lvl >= 2:  # noqa: PLR2004
            console_level = "DEBUG"
        case _:
            console_level = "WARNING"

    return {
        "version": 1,
        "disable_existing_loggers": False,
        "filters": {
            "run_id_filter": {
                "()": RunIdFilter,
            },
        },
        "formatters": {
            "human": {
                "()": ColorFormatter,
                "format": "%(asctime)s [%(levelname)s] %(name)s: %(message)s",
                "datefmt": "%H:%M:%S",
            },
            "json": {
                "()": "pythonjsonlogger.json.JsonFormatter",
                "fmt": "%(asctime)s %(levelname)s %(run_id)s %(name)s %(message)s",
                "rename_fields": {"asctime": "timestamp", "levelname": "level"},
            },
        },
        "handlers": {
            "console": {
                "class": "logging.StreamHandler",
                "formatter": "human",
                "level": console_level,
                "stream": "ext://sys.stderr",
            },
            "file": {
                "class": "logging.handlers.RotatingFileHandler",
                "formatter": "json",
                "level": "DEBUG",
                "filename": str(log_dir / "app.json.log"),
                "maxBytes": 10_485_760,  # 10 MB (Clean digit separators)
                "backupCount": 5,
                "encoding": "utf8",
                "filters": ["run_id_filter"],
            },
        },
        "loggers": {
            "citecraft": {
                "handlers": (
                    ["console"]
                    if os.environ.get("ENV") == "test"
                    else ["console", "file"]
                ),  # Avoid big files in testing phase
                "level": "DEBUG",
                "propagate": False,
            },
        },
    }


def setup_logging(verbose_level: int = 0) -> tuple[Path, Path, bool]:
    """Set up system loggers.

    Return log directory path, original intended path,
    and a boolean = True if it had to resort to a temp directory.
    """
    log_dir, intended_dir, is_fallback = get_safe_dir("logs")

    try:
        config = get_logging_config(log_dir, verbose_level=verbose_level)
        logging.config.dictConfig(config)
    except (ValueError, KeyError, ImportError, OSError) as e:
        # Fallback to standard error console if directory is locked or unwritable
        print(
            f"CRITICAL: Failed to initialize logging system configuration: {e}",
            file=sys.stderr,
        )
        traceback.print_exc(file=sys.stderr)
        logging.basicConfig(level=logging.WARNING, stream=sys.stderr)

    return log_dir, intended_dir, is_fallback

# tests/conftest.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Global test fixtures and testing configuration."""

import logging
from collections.abc import Generator
from pathlib import Path
from typing import NoReturn
from unittest.mock import patch

import pytest

from citecraft.network.http_client_registry import (
    get_http_client_registry,
)
from citecraft.utils import AppConfig, create_config, get_config


def pytest_collection_modifyitems(
    config: pytest.Config,
    items: list[pytest.Item],
) -> None:
    """Assign 'unit' marker to any test without 'e2e' or 'integration' markers."""
    for item in items:
        has_other_marker = any(
            item.get_closest_marker(name) for name in ["e2e", "integration"]
        )

        if not has_other_marker:
            item.add_marker(pytest.mark.unit)


@pytest.fixture(autouse=True)
def block_network_calls(monkeypatch: pytest.MonkeyPatch) -> None:
    """Guard rails to prevent any outbound network requests during testing."""

    def raised_error(*args: object, **kwargs: object) -> NoReturn:
        err_msg = "Network call attempted during isolated unit test execution."
        raise RuntimeError(err_msg)

    monkeypatch.setattr("socket.socket.connect", raised_error)


@pytest.fixture(autouse=True)
def _clear_config_cache() -> Generator[None, None, None]:
    """Clear the lru_cache of get_config before and after each test."""
    get_config.cache_clear()
    yield
    get_config.cache_clear()


@pytest.fixture(autouse=True)
def _clear_registry_cache() -> None:
    """Automatically clear the lru_cache of get_registry before each test."""
    get_http_client_registry.cache_clear()


@pytest.fixture(autouse=True)
def assert_logging_integrity(
    caplog: pytest.LogCaptureFixture,
) -> Generator[None, None, None]:
    """Verify log propagation integrity at the end of every test execution."""
    yield

    caplog.set_level(logging.INFO)
    canary_message = "LOGGING_INTEGRITY_CHECK_CANARY_TOKEN"

    logging.getLogger("citecraft").info(canary_message)

    if canary_message not in caplog.text:
        err_msg = (
            "CRITICAL: This test broke the global logging propagation! "
            "This usually happens when `setup_logging()` or `logging.basicConfig()` "
            "is called by the production code without being mocked. "
            "Please ensure you apply a proper patch/mock in this test script.",
        )
        raise AssertionError(err_msg)


@pytest.fixture
def test_config(tmp_path: Path) -> Generator[AppConfig, None, None]:
    """Provide a configuration isolated with temporary directories and databases."""
    config_instance = create_config()
    test_config_obj = config_instance.model_copy(
        update={
            "local_repo_dir_path": tmp_path,
            "log_dir_path": tmp_path,
            "output_dir_path": tmp_path,
            "db_filepath": tmp_path / config_instance.db_filename,
        },
    )

    with patch(
        "citecraft.utils.config.get_config",
        return_value=test_config_obj,
    ):
        yield test_config_obj

```

Please confirm reception before I continue.

---

---

# LLM

I have received and analyzed the configuration suite for your project, `citecraft`. 

I have registered the following architectural details in active memory:

1. **Project Definition (`pyproject.toml`)**: Standard Python 3.12.9 project packaged via Hatch, relying on `citeproc-py`, `pydantic-settings`, and `httpx`. Strict linting configuration is active via `ruff` with standard Python 3.12 targets and testing-specific rule bypasses.
2. **Environment Configuration (`src/citecraft/utils/config.py`)**: An `AppConfig` class built using `pydantic-settings`. It enforces directory safety limits, holds default limits/timeouts for Crossref, DOI, and OpenAlex APIs, and manages dynamic cache paths via a computed `db_filepath` property.
3. **Logging System (`src/citecraft/logging_config.py`)**: Set up with a custom `RunIdFilter` for tracing and a `ColorFormatter` for clean terminal outputs. Logs automatically route to standard error or JSON logs depending on the environment.
4. **Test Suite Rules (`tests/conftest.py`)**: Standardizes test categories to `unit` unless explicitly marked, aggressively blocks outbound network sockets to prevent leakage, and verifies logging propagation using a canary message assertion.

I am ready to evaluate and generate the unit tests according to these specifications. Please provide the details of your test request.

---

---

# ME

```
# src/citecraft/repositories/doi_repository.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Repository for fetching and sanitizing metadata records using DOI resolution."""

import json
import logging
from http import HTTPStatus
from typing import Any

import httpx

from citecraft.network import (
    HTTPClientRegistry,
    HTTPClientWrapper,
    get_http_client_registry,
)
from citecraft.schemas import DoiType
from citecraft.utils import AppConfig, get_config

logger = logging.getLogger(__name__)


class DoiRepository:
    """Handles metadata extraction and processing specific to DOI resources."""

    def __init__(
        self,
        config: AppConfig | None = None,
        registry: HTTPClientRegistry | None = None,
    ) -> None:
        self.config: AppConfig = config or get_config()
        registry = registry or get_http_client_registry()
        self.http_client_wrapper: HTTPClientWrapper = registry.get_client("doi")

    def get_metadata(self, doi: DoiType) -> dict[str, Any]:
        """Retrieve CSL-JSON metadata via DOI content negotiation.

        Ensures that metadata contains an attribute id or DOI, necessary for
        CSLReference validation.
        """
        headers = {"Accept": "application/vnd.citationstyles.csl+json"}
        url = self.config.doi_api_url.replace("{object_name}", str(doi))

        try:
            res, _ = self.http_client_wrapper.get(url, headers=headers)
            if res is None:
                return {}

            res.raise_for_status()
            csl_metadata = res.json()

            if not self._is_valid_csl(csl_metadata, doi):
                return {}

            self._apply_blacklists(doi, csl_metadata)

        except json.JSONDecodeError:
            logger.warning(
                "Invalid format for CSL-JSON: %s",
                doi,
                extra={
                    "status": "KO",
                    "event": "doi_csl_json_invalid_format",
                    "doi": doi,
                },
            )
            return {}
        except httpx.HTTPStatusError as e:
            if e.response.status_code == HTTPStatus.NOT_FOUND:
                logger.warning(
                    "DOI not found (%d) for CSL-JSON: %s",
                    HTTPStatus.NOT_FOUND,
                    doi,
                    extra={
                        "status": "KO",
                        "event": "doi_csl_json_not_found",
                        "doi": doi,
                        "status_code": HTTPStatus.NOT_FOUND,
                    },
                )
                return {}
            raise
        else:
            return csl_metadata

    def _is_valid_csl(self, csl_metadata: dict[str, Any], doi: DoiType) -> bool:
        """Validate that essential identifying keys exist within CSL-JSON record."""
        if (
            "id" not in csl_metadata
            and "DOI" not in csl_metadata
            and "doi" not in csl_metadata
        ):
            logger.debug(
                "Invalid CSL-JSON content: %s",
                json.dumps(csl_metadata, separators=(",", ":")),
                extra={
                    "status": "KO",
                    "event": "doi_csl_json_dump",
                    "doi": doi,
                },
            )
            logger.warning(
                "CSL-JSON metadata invalid (missing 'id' and 'DOI'/'doi' "
                "field) for DOI:%s",
                doi,
                extra={
                    "status": "KO",
                    "event": "doi_csl_json_invalid_id_and_doi",
                    "doi": doi,
                },
            )
            return False
        return True

    def _apply_blacklists(self, doi: DoiType, csl_metadata: dict[str, Any]) -> None:
        """Filter out blacklisted work attributes and author profile fields."""
        work_blacklist_fields = self.config.work_crossref_schema_blacklist_fields
        for field in work_blacklist_fields:
            csl_metadata.pop(field, None)

        author_blacklist_fields = self.config.author_crossref_schema_blacklist_fields
        if (
            author_blacklist_fields
            and "author" in csl_metadata
            and isinstance(csl_metadata["author"], list)
        ):
            for author_entry in csl_metadata["author"]:
                if isinstance(author_entry, dict):
                    for sub_field in author_blacklist_fields:
                        author_entry.pop(sub_field, None)

        logger.debug(
            "Successfully resolved CSL-JSON metadata for DOI: %s",
            doi,
            extra={
                "status": "OK",
                "event": "doi_csl_json_success",
                "doi": doi,
            },
        )

# tests/unit/repositories/test_doi_repository.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Unit tests for validating DOI metadata resolution and filtering operations."""

import json
from typing import Any
from unittest.mock import MagicMock, patch

import httpx
import pytest

from citecraft.repositories import DoiRepository
from citecraft.utils import AppConfig


@pytest.fixture
def repo(test_config: AppConfig) -> DoiRepository:
    """Provide DoiRepository testing instance."""
    return DoiRepository(config=test_config)


def test_get_metadata_success(repo: DoiRepository) -> None:
    """Verify that valid CSL-JSON metadata is mapped and returned correctly."""
    mock_json_data = {
        "id": "10.1000/182",
        "type": "article-journal",
        "title": "Title of the Paper",
        "author": [{"family": "Doe", "given": "J."}],
    }
    mock_response = MagicMock()
    mock_response.status_code = 200
    mock_response.json.return_value = mock_json_data

    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_response, None),
    ) as mock_get:
        result = repo.get_metadata("10.1000/182")

        assert result == mock_json_data
        assert isinstance(result, dict)

        _, kwargs = mock_get.call_args
        assert "headers" in kwargs
        assert kwargs["headers"]["Accept"] == "application/vnd.citationstyles.csl+json"


@pytest.mark.parametrize(
    ("side_effect", "json_val", "expected_log_sub"),
    [
        # Case A: Missing required keys (id and DOI)
        (
            None,
            {"type": "article-journal", "title": "Orphan Title"},
            "metadata invalid",
        ),
        # Case B: JSONDecodeError simulation
        (
            json.JSONDecodeError("Expecting value", "", 0),
            None,
            "Invalid format for CSL-JSON",
        ),
        # Case C: HTTP 404 Status Error simulation
        (
            httpx.HTTPStatusError(
                "404 Not Found",
                request=httpx.Request("GET", "https://doi.org/invalid/doi"),
                response=httpx.Response(404),
            ),
            None,
            "not found",
        ),
    ],
)
def test_get_metadata_safe_fallbacks(
    repo: DoiRepository,
    side_effect: Exception | None,
    json_val: dict[str, Any] | None,
    expected_log_sub: str,
) -> None:
    """Verify that errors fall back to empty records."""
    mock_response = MagicMock()
    mock_response.status_code = 200
    if json_val is not None:
        mock_response.json.return_value = json_val
    if isinstance(side_effect, json.JSONDecodeError):
        mock_response.json.side_effect = side_effect

    with (
        patch.object(repo.http_client_wrapper, "get") as mock_get,
        patch("citecraft.repositories.doi_repository.logger.warning") as mock_warn,
    ):
        if isinstance(side_effect, httpx.HTTPStatusError):
            mock_get.side_effect = side_effect
        else:
            mock_get.return_value = (mock_response, None)

        result = repo.get_metadata("10.1000/fallback-target")

        assert result == {}
        mock_warn.assert_called_once()

        log_content = " ".join(str(arg) for arg in mock_warn.call_args[0]).lower()
        assert expected_log_sub.lower() in log_content


def test_get_metadata_server_error_500_bubbles_up(repo: DoiRepository) -> None:
    """Verify that server-level errors (like 500) bubble up and raise immediately."""
    mock_request = httpx.Request("GET", "https://doi.org/10.1000/broken")
    mock_response = httpx.Response(status_code=500, request=mock_request)
    error = httpx.HTTPStatusError(
        "500 Internal Server Error",
        request=mock_request,
        response=mock_response,
    )

    with (
        patch.object(repo.http_client_wrapper, "get", side_effect=error),
        pytest.raises(httpx.HTTPStatusError),
    ):
        repo.get_metadata("10.1000/broken")


def test_get_metadata_applies_blacklists_successfully(test_config: AppConfig) -> None:
    """Verify that blacklists strip fields from work and author scopes."""
    test_config = test_config.model_copy(
        update={
            "work_crossref_schema_blacklist_fields": [
                "issns",
                "assertion",
                "is-referenced-by-count",
            ],
            "author_crossref_schema_blacklist_fields": [
                "ORCID",
                "authenticated-orcid",
                "role",
            ],
        },
    )
    repo = DoiRepository(config=test_config)

    mock_raw_csl = {
        "DOI": "10.1038/s41561-020-0585-2",
        "issns": ["1752-0894"],
        "is-referenced-by-count": 79,
        "assertion": [{"label": "Received"}],
        "title": "Steady erosion rates in the Himalayas",
        "author": [
            {
                "family": "Lenard",
                "given": "Sebastien J. P.",
                "ORCID": "https://orcid.org/0000-0003-3358-7197",
                "authenticated-orcid": False,
                "role": [{"role": "author"}],
            },
            {"family": "Lavé", "given": "Jérôme", "role": [{"role": "author"}]},
        ],
    }
    mock_response = MagicMock()
    mock_response.status_code = 200
    mock_response.json.return_value = mock_raw_csl

    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_response, None),
    ):
        filtered_result = repo.get_metadata("10.1038/s41561-020-0585-2")

        assert "DOI" in filtered_result
        assert "title" in filtered_result
        assert "issns" not in filtered_result
        assert "is-referenced-by-count" not in filtered_result
        assert "assertion" not in filtered_result

        assert "author" in filtered_result
        authors = filtered_result["author"]
        assert len(authors) == 2

        assert authors[0]["family"] == "Lenard"
        assert "ORCID" not in authors[0]
        assert "authenticated-orcid" not in authors[0]
        assert "role" not in authors[0]

        assert authors[1]["family"] == "Lavé"
        assert "role" not in authors[1]


def test_get_metadata_with_empty_or_missing_blacklists(test_config: AppConfig) -> None:
    """Verify that blacklisting logic passes through unchanged when scopes are empty."""
    test_config = test_config.model_copy(
        update={
            "work_crossref_schema_blacklist_fields": [],
            "author_crossref_schema_blacklist_fields": [],
        },
    )
    repo = DoiRepository(config=test_config)
    mock_json_data = {
        "id": "10.1000/182",
        "author": [{"family": "Doe", "ORCID": "0000-0001"}],
    }
    mock_response = MagicMock()
    mock_response.status_code = 200
    mock_response.json.return_value = mock_json_data

    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_response, None),
    ):
        result = repo.get_metadata("10.1000/182")
        assert result == mock_json_data
        assert "ORCID" in result["author"][0]


def test_is_valid_csl_delegate(repo: DoiRepository) -> None:
    """Directly test the structural validation helper rule."""
    valid_record = {"id": "10.1234/test"}
    invalid_record = {"title": "Missing ID"}

    assert repo._is_valid_csl(valid_record, "10.1234/test") is True
    assert repo._is_valid_csl(invalid_record, "10.1234/test") is False


def test_apply_blacklists_delegate(repo: DoiRepository) -> None:
    """Directly test that internal blacklist manipulation strips keys safely."""
    repo.config = repo.config.model_copy(
        update={
            "work_crossref_schema_blacklist_fields": ["unwanted_work_key"],
            "author_crossref_schema_blacklist_fields": ["unwanted_author_key"],
        },
    )
    doi = "10.1111/test"
    payload = {
        "id": "10.1111/test",
        "unwanted_work_key": "delete_me",
        "author": [{"family": "Smith", "unwanted_author_key": "strip_me"}],
    }

    repo._apply_blacklists(doi, payload)

    assert "unwanted_work_key" not in payload
    assert "unwanted_author_key" not in payload["author"][0]
```

The tests don't cover these branches:

- in get_metadata(), if res is None == True. This may be because http_client_wrapper is not mocked
- in _apply_blacklists(), if isinstance(author_entry, dict) == False.

Please give the updated test script.

---

---

# LLM

```
# tests/unit/repositories/test_doi_repository.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Unit tests for validating DOI metadata resolution and filtering operations."""

import json
from typing import Any
from unittest.mock import MagicMock, patch

import httpx
import pytest

from citecraft.repositories import DoiRepository
from citecraft.utils import AppConfig


@pytest.fixture
def repo(test_config: AppConfig) -> DoiRepository:
    """Provide DoiRepository testing instance."""
    return DoiRepository(config=test_config)


def test_get_metadata_success(repo: DoiRepository) -> None:
    """Verify that valid CSL-JSON metadata is mapped and returned correctly."""
    mock_json_data = {
        "id": "10.1000/182",
        "type": "article-journal",
        "title": "Title of the Paper",
        "author": [{"family": "Doe", "given": "J."}],
    }
    mock_response = MagicMock()
    mock_response.status_code = 200
    mock_response.json.return_value = mock_json_data

    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_response, None),
    ) as mock_get:
        result = repo.get_metadata("10.1000/182")

        assert result == mock_json_data
        assert isinstance(result, dict)

        _, kwargs = mock_get.call_args
        assert "headers" in kwargs
        assert kwargs["headers"]["Accept"] == "application/vnd.citationstyles.csl+json"


def test_get_metadata_client_returns_none(repo: DoiRepository) -> None:
    """Verify that get_metadata returns an empty dictionary when client returns None."""
    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(None, None),
    ):
        result = repo.get_metadata("10.1000/none-response")
        assert result == {}


@pytest.mark.parametrize(
    ("side_effect", "json_val", "expected_log_sub"),
    [
        # Case A: Missing required keys (id and DOI)
        (
            None,
            {"type": "article-journal", "title": "Orphan Title"},
            "metadata invalid",
        ),
        # Case B: JSONDecodeError simulation
        (
            json.JSONDecodeError("Expecting value", "", 0),
            None,
            "Invalid format for CSL-JSON",
        ),
        # Case C: HTTP 404 Status Error simulation
        (
            httpx.HTTPStatusError(
                "404 Not Found",
                request=httpx.Request("GET", "https://doi.org/invalid/doi"),
                response=httpx.Response(404),
            ),
            None,
            "not found",
        ),
    ],
)
def test_get_metadata_safe_fallbacks(
    repo: DoiRepository,
    side_effect: Exception | None,
    json_val: dict[str, Any] | None,
    expected_log_sub: str,
) -> None:
    """Verify that errors fall back to empty records."""
    mock_response = MagicMock()
    mock_response.status_code = 200
    if json_val is not None:
        mock_response.json.return_value = json_val
    if isinstance(side_effect, json.JSONDecodeError):
        mock_response.json.side_effect = side_effect

    with (
        patch.object(repo.http_client_wrapper, "get") as mock_get,
        patch("citecraft.repositories.doi_repository.logger.warning") as mock_warn,
    ):
        if isinstance(side_effect, httpx.HTTPStatusError):
            mock_get.side_effect = side_effect
        else:
            mock_get.return_value = (mock_response, None)

        result = repo.get_metadata("10.1000/fallback-target")

        assert result == {}
        mock_warn.assert_called_once()

        log_content = " ".join(str(arg) for arg in mock_warn.call_args[0]).lower()
        assert expected_log_sub.lower() in log_content


def test_get_metadata_server_error_500_bubbles_up(repo: DoiRepository) -> None:
    """Verify that server-level errors (like 500) bubble up and raise immediately."""
    mock_request = httpx.Request("GET", "https://doi.org/10.1000/broken")
    mock_response = httpx.Response(status_code=500, request=mock_request)
    error = httpx.HTTPStatusError(
        "500 Internal Server Error",
        request=mock_request,
        response=mock_response,
    )

    with (
        patch.object(repo.http_client_wrapper, "get", side_effect=error),
        pytest.raises(httpx.HTTPStatusError),
    ):
        repo.get_metadata("10.1000/broken")


def test_get_metadata_applies_blacklists_successfully(test_config: AppConfig) -> None:
    """Verify that blacklists strip fields from work and author scopes."""
    test_config = test_config.model_copy(
        update={
            "work_crossref_schema_blacklist_fields": [
                "issns",
                "assertion",
                "is-referenced-by-count",
            ],
            "author_crossref_schema_blacklist_fields": [
                "ORCID",
                "authenticated-orcid",
                "role",
            ],
        },
    )
    repo = DoiRepository(config=test_config)

    mock_raw_csl = {
        "DOI": "10.1038/s41561-020-0585-2",
        "issns": ["1752-0894"],
        "is-referenced-by-count": 79,
        "assertion": [{"label": "Received"}],
        "title": "Steady erosion rates in the Himalayas",
        "author": [
            {
                "family": "Lenard",
                "given": "Sebastien J. P.",
                "ORCID": "https://orcid.org/0000-0003-3358-7197",
                "authenticated-orcid": False,
                "role": [{"role": "author"}],
            },
            {"family": "Lavé", "given": "Jérôme", "role": [{"role": "author"}]},
        ],
    }
    mock_response = MagicMock()
    mock_response.status_code = 200
    mock_response.json.return_value = mock_raw_csl

    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_response, None),
    ):
        filtered_result = repo.get_metadata("10.1038/s41561-020-0585-2")

        assert "DOI" in filtered_result
        assert "title" in filtered_result
        assert "issns" not in filtered_result
        assert "is-referenced-by-count" not in filtered_result
        assert "assertion" not in filtered_result

        assert "author" in filtered_result
        authors = filtered_result["author"]
        assert len(authors) == 2

        assert authors[0]["family"] == "Lenard"
        assert "ORCID" not in authors[0]
        assert "authenticated-orcid" not in authors[0]
        assert "role" not in authors[0]

        assert authors[1]["family"] == "Lavé"
        assert "role" not in authors[1]


def test_get_metadata_with_empty_or_missing_blacklists(test_config: AppConfig) -> None:
    """Verify that blacklisting logic passes through unchanged when scopes are empty."""
    test_config = test_config.model_copy(
        update={
            "work_crossref_schema_blacklist_fields": [],
            "author_crossref_schema_blacklist_fields": [],
        },
    )
    repo = DoiRepository(config=test_config)
    mock_json_data = {
        "id": "10.1000/182",
        "author": [{"family": "Doe", "ORCID": "0000-0001"}],
    }
    mock_response = MagicMock()
    mock_response.status_code = 200
    mock_response.json.return_value = mock_json_data

    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_response, None),
    ):
        result = repo.get_metadata("10.1000/182")
        assert result == mock_json_data
        assert "ORCID" in result["author"][0]


def test_is_valid_csl_delegate(repo: DoiRepository) -> None:
    """Directly test the structural validation helper rule."""
    valid_record = {"id": "10.1234/test"}
    invalid_record = {"title": "Missing ID"}

    assert repo._is_valid_csl(valid_record, "10.1234/test") is True
    assert repo._is_valid_csl(invalid_record, "10.1234/test") is False


def test_apply_blacklists_delegate(repo: DoiRepository) -> None:
    """Directly test that internal blacklist manipulation strips keys safely."""
    repo.config = repo.config.model_copy(
        update={
            "work_crossref_schema_blacklist_fields": ["unwanted_work_key"],
            "author_crossref_schema_blacklist_fields": ["unwanted_author_key"],
        },
    )
    doi = "10.1111/test"
    payload = {
        "id": "10.1111/test",
        "unwanted_work_key": "delete_me",
        "author": [{"family": "Smith", "unwanted_author_key": "strip_me"}],
    }

    repo._apply_blacklists(doi, payload)

    assert "unwanted_work_key" not in payload
    assert "unwanted_author_key" not in payload["author"][0]


def test_apply_blacklists_with_non_dict_author_entry(test_config: AppConfig) -> None:
    """Verify that author blacklisting bypasses non-dictionary author elements safely."""
    test_config = test_config.model_copy(
        update={
            "author_crossref_schema_blacklist_fields": ["unwanted_author_key"],
        },
    )
    repo = DoiRepository(config=test_config)
    doi = "10.1111/test"
    payload = {
        "id": "10.1111/test",
        "author": [
            "Literal string author representation",
            {"family": "Smith", "unwanted_author_key": "strip_me"},
        ],
    }

    repo._apply_blacklists(doi, payload)

    assert payload["author"][0] == "Literal string author representation"
    assert isinstance(payload["author"][1], dict)
    assert "unwanted_author_key" not in payload["author"][1]
```

---

---

# ME

```
# src/citecraft/repositories/journal_repository.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Repository for tracking, fetching, and updating local academic journal metadata."""

import logging
import time
from dataclasses import dataclass
from datetime import UTC, date, datetime, timedelta
from typing import Any, Literal

from citecraft.parsers import JournalParser
from citecraft.schemas import JournalMetadata
from citecraft.utils import AppConfig

from .base_repository import BaseRepository

logger = logging.getLogger(__name__)


@dataclass(slots=True)
class UpdateBatchState:
    """Tracks operational state across batch execution segments."""

    total: int
    processed: int = 0
    update_count: int = 0
    last_heartbeat: float = 0.0


class JournalRepository(BaseRepository[JournalMetadata]):
    """Handles persistence and retrieval workflows for journal metadata records."""

    def __init__(
        self,
        local_filename: str = "journal_records.json",
        config: AppConfig | None = None,
        api: str = "crossref",
    ) -> None:
        super().__init__(
            local_filename,
            model_class=JournalMetadata,
            config=config,
            api=api,
        )
        self.has_pending_updates: bool = False

    def _log_heartbeat_if_needed(
        self,
        processed: int,
        total: int,
        last_time: float,
    ) -> float:
        """Log update batch progress status every 10 seconds of processing time."""
        current_time = time.time()
        if (
            current_time - last_time
            > self.config.default_logging_frequency_for_batch_updates
        ):
            remaining = total - processed
            logger.info(
                "Batch update status: %d updates remaining out of %d",
                remaining,
                total,
                extra={
                    "status": "OK",
                    "event": "journal_update_heartbeat",
                    "remaining_count": remaining,
                    "total_count": total,
                },
            )
            return current_time
        return last_time

    def get_issns_by_input_title(self, input_title: str) -> list[str]:
        """Return unique, non-null ISSNs present in records for a specific title."""
        return sorted(
            {
                r.issn
                for r in self.records
                if r.input_title == input_title and r.issn is not None
            },
        )

    def get_unique_issns_for_titles(self, titles: list[str]) -> list[str]:
        """Extract unique, non-null ISSNs for a validated list of titles."""
        if not titles:
            return []

        # Normalize and put in a set (O(1) search)
        required_normalized = {JournalParser.normalize_title(t) for t in titles if t}

        # Cache filter (O(N))
        unique_issns: set[str] = set()

        for record in self.records:
            if (
                record.issn
                and JournalParser.normalize_title(record.input_title)
                in required_normalized
            ):
                unique_issns.add(record.issn)

        return sorted(unique_issns)

    def _fetch_crossref_items(self, input_title: str) -> list[dict[str, Any]] | None:
        """Fetch matching raw journal records from Crossref API."""
        params = {
            "query": input_title,
            "rows": 200,
            "mailto": self.config.user_email,
        }
        response, predicted_url = self.http_client_wrapper.get(
            str(self.config.crossref_api_journals_url),
            params=params,
            headers=self.headers,
        )
        if response is None:
            logger.warning(
                "Empty journal '%s' response from URL %s.",
                input_title,
                predicted_url,
                extra={
                    "status": "KO",
                    "event": "empty_journal_response_from_url",
                    "input_title": input_title,
                    "predicted_url": predicted_url,
                },
            )
            return None

        response.raise_for_status()
        return response.json().get("message", {}).get("items", [])

    def _filter_matches(
        self,
        input_title: str,
        items: list[dict[str, Any]],
    ) -> tuple[list[dict[str, Any]], list[str] | None]:
        """Classify Crossref items into exact matches or similar matches."""
        # Filter exact title representations
        exact_matches = [
            item for item in items if item.get("title", "").strip() == input_title
        ]

        if exact_matches:
            # Discard exact matches other than the 1st one
            if len(exact_matches) > 1:
                logger.warning(
                    "Discarded %d duplicate titles in the repo for journal: %s",
                    len(exact_matches),
                    input_title,
                    extra={
                        "status": "WARNING",
                        "event": "crossref_duplicate_titles_discarded",
                        "discarded_count": len(exact_matches),
                        "input_title": input_title,
                    },
                )
            return [exact_matches[0]], None

        # Look for potential matches based on flexible business rules
        target_norm = JournalParser.normalize_title(input_title)
        similar_items = []
        raw_similar_titles = []

        for item in items:
            raw_title = item.get("title", "").strip()
            if raw_title and JournalParser.normalize_title(raw_title) == target_norm:
                similar_items.append(item)
                raw_similar_titles.append(raw_title)

        if similar_items:
            # Deduplicate titles keeping order
            similar_titles = list(dict.fromkeys(raw_similar_titles))
            logger.warning(
                "Journal %s not found. Use similar titles to fill metadata: %s",
                input_title,
                ", ".join(similar_titles),
                extra={
                    "status": "WARNING",
                    "event": "crossref_journal_similar_found",
                    "input_title": input_title,
                    "similar_titles": similar_titles,
                },
            )
            return similar_items, similar_titles

        return [], None

    def _resolve_publication_years(self, issn: str) -> tuple[int | None, int | None]:
        """Resolve start and end publication years for a given ISSN."""
        min_year = self.get_issn_year_endpoint(issn, "asc")
        max_year = self.get_issn_year_endpoint(issn, "desc")
        return min_year, max_year

    def _build_metadata_records(
        self,
        input_title: str,
        working_items: list[dict[str, Any]],
        similar_titles: list[str] | None,
    ) -> list[JournalMetadata]:
        """Map working Crossref metadata items to list of JournalMetadata."""
        journal_records: list[JournalMetadata] = []
        processed_issns: set[str] = set()

        for item in working_items:
            true_title = item.get("title", "")
            publisher = item.get("publisher", "")
            # remove duplicate ISSNs, get MUST be on ISSN.
            raw_issns = item.get("ISSN") or item.get("issn") or []
            issns = list(dict.fromkeys(raw_issns))

            if not issns:
                logger.warning(
                    "Journal %s found but contains no ISSN metadata.",
                    true_title,
                    extra={
                        "status": "WARNING",
                        "event": "crossref_journal_missing_issn",
                        "input_title": input_title,
                        "true_title": true_title,
                    },
                )
                journal_records.append(
                    JournalMetadata(
                        input_title=input_title,
                        true_title=true_title,
                        publisher=publisher,
                        issn=None,
                        start_year=None,
                        end_year=None,
                        similar_titles=similar_titles,
                    ),
                )
                continue

            for issn in issns:
                if issn in processed_issns:
                    logger.debug(
                        "ISSN %s already processed in this batch. Skipping.",
                        issn,
                    )
                    continue

                processed_issns.add(issn)
                logger.info(
                    "Retrieving %s / %s publication range...",
                    input_title,
                    issn,
                    extra={
                        "status": "OK",
                        "event": "crossref_issn_range_query_start",
                        "input_title": input_title,
                        "issn": issn,
                    },
                )

                start_year, end_year = self._resolve_publication_years(issn)

                if start_year is None or end_year is None:
                    logger.warning(
                        "Journal %s / %s has no published work found. "
                        "Keeping record with empty dates.",
                        input_title,
                        issn,
                        extra={
                            "status": "WARNING",
                            "event": "crossref_journal_no_works_found",
                            "input_title": input_title,
                            "issn": issn,
                        },
                    )
                    start_year = None
                    end_year = None

                journal_records.append(
                    JournalMetadata(
                        input_title=input_title,
                        true_title=true_title,
                        publisher=publisher,
                        issn=issn,
                        start_year=start_year,
                        end_year=end_year,
                        similar_titles=similar_titles,
                    ),
                )

        return journal_records

    def get_journal_metadata(self, input_title: str) -> list[JournalMetadata]:
        """Retrieve and process journal metadata maps from the remote Crossref API.

        Get journal metadata filtered on the exact match or similar matches of the title
        (input_title). A title can correspond to several
        records or none, each of them identified by a unique ISSN.
        - Case exact match: Returns a list of JournalMetadata corresponding to
            the first exact title found (one by distinct ISSN).
        - Case no exact match but similar titles found: Resolves ISSNs for all similar
            titles, queries their publication ranges, and returns JournalMetadata
            records mapped to the original input_title and the similar_titles list.
        - Case no exact match: Returns a list of one incomplete JournalMetadata only.
        """
        logger.info(
            "Retrieving journal %s metadata from Crossref...",
            input_title,
            extra={
                "status": "OK",
                "event": "crossref_journal_query_start",
                "input_title": input_title,
            },
        )
        items = self._fetch_crossref_items(input_title)
        if items is None:
            return [JournalMetadata(input_title=input_title)]

        working_items, similar_titles = self._filter_matches(input_title, items)
        if not working_items:
            logger.warning(
                "Journal %s not found.",
                input_title,
                extra={
                    "status": "KO",
                    "event": "crossref_journal_not_found",
                    "input_title": input_title,
                },
            )
            return [JournalMetadata(input_title=input_title)]

        return self._build_metadata_records(input_title, working_items, similar_titles)

    def get_issn_year_endpoint(
        self,
        issn: str,
        order: Literal["asc", "desc"],
    ) -> int | None:
        """Get publication endpoints based on print/online works for the ISSN.

        asc: oldest / desc: youngest.
        """
        params = {
            "sort": "published",
            "order": order,
            "rows": 1,
            "mailto": self.config.user_email,
        }
        url = self.config.crossref_api_journals_issn_url.replace(
            "{object_name}",
            str(issn),
        )

        response, predicted_url = self.http_client_wrapper.get(
            url,
            params=params,
            headers=self.headers,
        )

        if response is None:
            logger.warning(
                "Empty journal issn '%s' response from URL %s.",
                issn,
                predicted_url,
                extra={
                    "status": "KO",
                    "event": "empty_journal_response_from_url",
                    "issn": issn,
                    "predicted_url": predicted_url,
                },
            )
            return None

        response.raise_for_status()
        items = response.json().get("message", {}).get("items", [])
        if not items:
            return None

        work = items[0]
        p_date = work.get("published-print", {}).get("date-parts", [[None]])[0][0]
        o_date = work.get("published-online", {}).get("date-parts", [[None]])[0][0]

        # Earliest or latest year found between print/online
        years = [y for y in [p_date, o_date] if y is not None]
        if not years:
            return None
        return min(years) if order == "asc" else max(years)

    def get_sync_status(self) -> dict[str, int | bool]:
        """Analyze local records completeness and update status against thresholds.

        Returns a status dictionary useful for control flow in core.py.
        TODO: Check if we still need this method (after the development of catching
        metadata from similar titles)
        """
        expiration_date = datetime.now(UTC).date() - timedelta(
            days=self.config.journal_update_days,
        )
        missing_count = 0
        expired_count = 0

        for record in self.records:
            if not record.is_complete:
                missing_count += 1
            else:
                last_update = date.fromisoformat(record.update)
                if last_update < expiration_date:
                    expired_count += 1

        return {
            "is_fully_synchronized": missing_count == 0 and expired_count == 0,
            "missing_metadata_count": missing_count,
            "expired_metadata_count": expired_count,
            "has_pending_updates": self.has_pending_updates,
        }

    def _partition_records(
        self,
        expiration_date: date,
    ) -> tuple[list[JournalMetadata], list[JournalMetadata], list[JournalMetadata]]:
        """Partition local records into missing, expired, and valid categories."""
        missing: list[JournalMetadata] = []
        expired: list[JournalMetadata] = []
        valid: list[JournalMetadata] = []

        for record in self.records:
            last_update = date.fromisoformat(record.update)

            if not record.is_complete:
                missing.append(record)
            elif last_update < expiration_date:
                expired.append(record)
            else:
                valid.append(record)

        return missing, expired, valid

    def _update_batch(
        self,
        targets: list[JournalMetadata],
        new_records: list[JournalMetadata],
        state: UpdateBatchState,
    ) -> None:
        """Process a collection of target records up to the configured limit."""
        for record in targets:
            if state.update_count < self.config.journal_update_limit:
                new_data = self.get_journal_metadata(record.input_title)
                if new_data:
                    new_records.extend(new_data)
                else:
                    new_records.append(record)
                state.update_count += 1
            else:
                new_records.append(record)

            state.processed += 1
            state.last_heartbeat = self._log_heartbeat_if_needed(
                state.processed,
                state.total,
                state.last_heartbeat,
            )

    def update_all(self) -> None:
        """Sync and update outdated journal records up to configured limits.

        Update the records missing metadata (Priority 1) and records with expired
        metadata (Priority 2) with up-to-date metatata from the remote repo.
        Warning: Update restricted to a max number of journals, doesn't include
        regular local saving of the updates.
        """
        expiration_date = datetime.now(UTC).date() - timedelta(
            days=self.config.journal_update_days,
        )
        logger.info(
            "Updating journals without metadata or metadata older than: %s",
            str(expiration_date),
            extra={
                "status": "OK",
                "event": "journal_update_process_started",
                "expiration_threshold": str(expiration_date),
            },
        )

        missing, expired, valid = self._partition_records(
            expiration_date,
        )  # missing = missing metadata

        logger.info(
            "Journal categorization completed. Missing: %d, Expired: %d, Valid: %d",
            len(missing),
            len(expired),
            len(valid),
            extra={
                "status": "OK",
                "event": "journal_update_categorization",
                "missing_count": len(missing),
                "expired_count": len(expired),
                "valid_count": len(valid),
            },
        )

        new_records: list[JournalMetadata] = list(valid)
        total_initial_targets = len(missing) + len(expired)
        state = UpdateBatchState(
            total=total_initial_targets,
            last_heartbeat=time.time(),
        )

        # Process Missing Metadata (Priority 1)
        self._update_batch(missing, new_records, state)

        # Process Expired Metadata (Priority 2)
        self._update_batch(expired, new_records, state)

        # Compute if any record that required update got skipped due to limit
        skipped_count = total_initial_targets - state.update_count
        self.has_pending_updates = skipped_count > 0

        if self.has_pending_updates:
            logger.warning(
                "Journal update limit reached. %d records still need updating.",
                skipped_count,
                extra={
                    "status": "WARNING",
                    "event": "journal_update_limit_reached",
                    "remaining_count": skipped_count,
                    "limit_configured": self.config.journal_update_limit,
                },
            )

        # Sort records: 1. Core metadata complete first, then incomplete.
        # 2. Alphabetical by input_title
        new_records.sort(key=lambda r: (not r.is_complete, r.input_title.lower()))
        self.records = new_records

        logger.info(
            "Journal metadata sync finalized. Updated %d journals",
            len(self.records),
            extra={
                "status": "OK",
                "event": "journal_update_completed",
                "total_records": len(self.records),
            },
        )

    def merge_new_titles(self, input_titles: list[str]) -> None:
        """Merge fresh titles as empty templates without introducing duplicates."""
        input_titles = list(dict.fromkeys(input_titles))
        existing_titles = {info.input_title for info in self.records}

        new_entries = [
            JournalMetadata(input_title=title)
            for title in input_titles
            if title not in existing_titles
        ]

        self.records.extend(new_entries)
        logger.info(
            "Merged %d new journal records into the local repository list",
            len(new_entries),
            extra={
                "status": "OK",
                "event": "journal_records_merged",
                "new_entries_count": len(new_entries),
            },
        )

# tests/unit/repositories/test_journal_repository.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Unit tests for the local journal metadata repository persistence workflows."""

import logging
import time
from datetime import UTC, datetime, timedelta
from typing import Any, Literal
from unittest.mock import MagicMock, patch

import pytest

from citecraft.repositories import JournalRepository
from citecraft.repositories.journal_repository import UpdateBatchState
from citecraft.schemas import JournalMetadata
from citecraft.utils import AppConfig


@pytest.fixture
def repo(test_config: AppConfig) -> JournalRepository:
    """Provide JournalRepository instance utilizing test config."""
    return JournalRepository(config=test_config)


def test_log_heartbeat_if_needed(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify heartbeat logger triggers periodically based on config threshold."""
    repo.config = repo.config.model_copy(
        update={"default_logging_frequency_for_batch_updates": 0.05},
    )
    with caplog.at_level(logging.INFO):
        t1 = time.time() - 0.1
        t2 = repo._log_heartbeat_if_needed(3, 10, t1)
        assert t2 > t1
        assert "Batch update status: 7 updates remaining" in caplog.text

        caplog.clear()
        t3 = repo._log_heartbeat_if_needed(4, 10, t2)
        assert t3 == t2
        assert "Batch update status" not in caplog.text


def test_get_journal_metadata_success(repo: JournalRepository) -> None:
    """Verify successful retrieval of ISSNs and years from endpoints."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {"items": [{"title": "Geology", "ISSN": ["0091-7613"]}]},
    }

    mock_year = MagicMock(status_code=200)
    mock_year.json.return_value = {
        "message": {
            "items": [
                {
                    "published-print": {"date-parts": [[1973]]},
                    "published-online": {"date-parts": [[1995]]},
                },
            ],
        },
    }

    with patch.object(repo.http_client_wrapper, "get") as mock_get:
        mock_get.side_effect = [
            (mock_main, None),
            (mock_year, None),
            (mock_year, None),
        ]

        results = repo.get_journal_metadata("Geology")

        assert len(results) == 1
        assert results[0].issn == "0091-7613"
        assert results[0].start_year == 1973
        assert results[0].end_year == 1995


def test_get_journal_metadata_not_found_behavior(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify fallback to template when no match is found."""
    mock_resp = MagicMock(status_code=200)
    mock_resp.json.return_value = {"message": {"items": []}}

    with (
        patch.object(repo.http_client_wrapper, "get", return_value=(mock_resp, None)),
        caplog.at_level(logging.WARNING),
    ):
        results = repo.get_journal_metadata("Unknown Journal")

        assert "Journal Unknown Journal not found." in caplog.text
        assert len(results) == 1
        assert results[0].issn is None


def test_get_journal_metadata_empty_response(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify fallback when network or API returns a null response."""
    with (
        patch.object(repo.http_client_wrapper, "get", return_value=(None, "mock_url")),
        caplog.at_level(logging.WARNING),
    ):
        results = repo.get_journal_metadata("Some Title")
        assert len(results) == 1
        assert results[0].input_title == "Some Title"
        assert results[0].issn is None
        assert "Empty journal 'Some Title' response from URL" in caplog.text


def test_get_journal_metadata_exact_matches_duplicate_discarded(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify multiple exact matches logs a warning and selects the first match."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {
            "items": [
                {"title": "Duplicate", "ISSN": ["0028-0836"]},
                {"title": "Duplicate", "ISSN": ["8765-4321"]},
            ],
        },
    }
    mock_year = MagicMock(status_code=200)
    mock_year.json.return_value = {
        "message": {"items": [{"published-print": {"date-parts": [[2001]]}}]},
    }

    with (
        patch.object(repo.http_client_wrapper, "get") as mock_get,
        caplog.at_level(logging.WARNING),
    ):
        mock_get.side_effect = [
            (mock_main, None),
            (mock_year, None),
            (mock_year, None),
        ]
        results = repo.get_journal_metadata("Duplicate")
        assert len(results) == 1
        assert results[0].issn == "0028-0836"
        assert "Discarded 2 duplicate titles" in caplog.text


def test_get_journal_metadata_similar_matches_found(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify fuzzy title similarity, filtering, and mapping logic."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {
            "items": [
                {
                    "title": "Nature Geoscience",
                    "publisher": "Springer Nature",
                    "issn": ["1752-0894"],
                },
                {
                    "title": "Nature-Geosciences",
                    "publisher": "Springer Nature",
                    "issn": ["1752-0894", "1752-0908"],
                },
                {
                    "title": "Nature Geoscience",
                    "publisher": "Springer Nature",
                    "issn": ["1752-0894"],
                },
            ],
        },
    }
    mock_year = MagicMock(status_code=200)
    mock_year.json.return_value = {
        "message": {
            "items": [
                {
                    "published-print": {"date-parts": [[2008]]},
                    "published-online": {"date-parts": [[2026]]},
                },
            ],
        },
    }

    with patch.object(repo.http_client_wrapper, "get") as mock_get:
        mock_get.side_effect = [
            (mock_main, None),
            (mock_year, None),
            (mock_year, None),
            (mock_year, None),
            (mock_year, None),
        ]

        with caplog.at_level(logging.WARNING):
            results = repo.get_journal_metadata("nature geoscience")

            assert "Use similar titles to fill metadata" in caplog.text
            assert len(results) == 2
            assert results[0].issn == "1752-0894"
            assert results[1].issn == "1752-0908"


def test_get_journal_metadata_missing_issn_handling(repo: JournalRepository) -> None:
    """Verify journals without ISSN are logged and kept without range queries."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {
            "items": [
                {
                    "title": "Natural Hazards",
                    "publisher": "Copernicus Publications",
                },
            ],
        },
    }

    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_main, None),
    ) as mock_get:
        results = repo.get_journal_metadata("Natural Hazards")

        assert len(results) == 1
        assert results[0].issn is None
        assert mock_get.call_count == 1


def test_get_journal_metadata_empty_publication_years(repo: JournalRepository) -> None:
    """Verify default empty year ranges when no published works are detected."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {
            "items": [
                {
                    "title": "Empty Journal",
                    "publisher": "Silent Publisher",
                    "issn": ["0010-051x"],
                },
            ],
        },
    }
    mock_year_empty = MagicMock(status_code=200)
    mock_year_empty.json.return_value = {"message": {"items": []}}

    with patch.object(repo.http_client_wrapper, "get") as mock_get:
        mock_get.side_effect = [
            (mock_main, None),
            (mock_year_empty, None),
            (mock_year_empty, None),
        ]

        results = repo.get_journal_metadata("Empty Journal")

        assert len(results) == 1
        assert results[0].start_year is None
        assert results[0].end_year is None
        assert mock_get.call_count == 3


def test_get_issns_by_input_title(repo: JournalRepository) -> None:
    """Verify unicity and extraction of valid ISSNs."""
    repo.records = [
        JournalMetadata(
            input_title="Journal A",
            issn="0010-051x",
            start_year=2000,
            end_year=2010,
        ),
        JournalMetadata(
            input_title="Journal A",
            issn="2049-3630",
            start_year=2005,
            end_year=2015,
        ),
        JournalMetadata(input_title="Journal A", issn=None),
    ]

    assert repo.get_issns_by_input_title("Journal A") == ["0010-051x", "2049-3630"]
    assert repo.get_issns_by_input_title("Non Existent") == []


def test_get_unique_issns_for_titles(repo: JournalRepository) -> None:
    """Verify unique sorting and extraction of ISSNs with normalization rules."""
    repo.records = [
        JournalMetadata(
            input_title="Nature Geoscience",
            issn="1752-0894",
            start_year=2008,
            end_year=2026,
        ),
        JournalMetadata(
            input_title="Journal of Climate",
            issn="2049-3630",
            start_year=2000,
            end_year=2010,
        ),
    ]

    assert repo.get_unique_issns_for_titles([]) == []
    assert repo.get_unique_issns_for_titles(["Unknown"]) == []
    assert repo.get_unique_issns_for_titles(
        ["Nature Geoscience", "Journal of Climate"],
    ) == ["1752-0894", "2049-3630"]


@pytest.mark.parametrize(
    ("order", "mock_items", "expected_year"),
    [
        (
            "asc",
            [
                {
                    "published-print": {"date-parts": [[1990]]},
                    "published-online": {"date-parts": [[2024]]},
                },
            ],
            1990,
        ),
        (
            "desc",
            [
                {
                    "published-print": {"date-parts": [[1990]]},
                    "published-online": {"date-parts": [[2024]]},
                },
            ],
            2024,
        ),
        ("asc", [], None),
        (
            "asc",
            [
                {
                    "published-print": {"date-parts": [[None]]},
                    "published-online": {"date-parts": [[2010]]},
                },
            ],
            2010,
        ),
    ],
)
def test_get_issn_year_endpoint_scenarios(
    repo: JournalRepository,
    order: Literal["asc", "desc"],
    mock_items: list[dict[str, Any]],
    expected_year: int | None,
) -> None:
    """Verify endpoint resolution behaves correctly under different scenarios."""
    mock_response = MagicMock()
    mock_response.json.return_value = {"message": {"items": mock_items}}

    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_response, None),
    ) as mock_get:
        assert repo.get_issn_year_endpoint("2049-3630", order) == expected_year
        mock_get.assert_called_once()


def test_get_issn_year_endpoint_empty_response(repo: JournalRepository) -> None:
    """Verify endpoint recovery when response resolves to None."""
    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(None, "mock_url"),
    ) as mock_get:
        assert repo.get_issn_year_endpoint("1234-5678", "asc") is None
        mock_get.assert_called_once()


def test_get_issn_year_endpoint_no_valid_years(repo: JournalRepository) -> None:
    """Verify endpoint resolution when payload exists but contains only null dates."""
    mock_response = MagicMock()
    mock_response.json.return_value = {
        "message": {
            "items": [
                {
                    "published-print": {"date-parts": [[None]]},
                    "published-online": {"date-parts": [[None]]},
                },
            ],
        },
    }
    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_response, None),
    ):
        assert repo.get_issn_year_endpoint("2049-3630", "asc") is None


@pytest.mark.parametrize(
    (
        "records",
        "pending_updates",
        "expected_synchronized",
        "expected_missing",
        "expected_expired",
        "expected_pending",
    ),
    [
        (
            [
                JournalMetadata(
                    input_title="Valid J",
                    true_title="T",
                    publisher="P",
                    issn="2049-3630",
                    start_year=2000,
                    end_year=2026,
                    update=str(
                        datetime.now(UTC).date() - timedelta(days=5),
                    ),
                ),
                JournalMetadata(
                    input_title="Expired J",
                    true_title="T",
                    publisher="P",
                    issn="1752-0908",
                    start_year=2000,
                    end_year=2026,
                    update=str(datetime.now(UTC).date() - timedelta(days=45)),
                ),
                JournalMetadata(
                    input_title="Missing J",
                    true_title=None,
                    publisher=None,
                    issn=None,
                    start_year=None,
                    end_year=None,
                    update=str(datetime.now(UTC).date() - timedelta(days=5)),
                ),
            ],
            True,
            False,
            1,
            1,
            True,
        ),
        (
            [
                JournalMetadata(
                    input_title="Valid J",
                    true_title="T",
                    publisher="P",
                    issn="2049-3630",
                    start_year=2000,
                    end_year=2026,
                    update=str(datetime.now(UTC).date() - timedelta(days=5)),
                ),
            ],
            False,
            True,
            0,
            0,
            False,
        ),
    ],
)
def test_get_sync_status_scenarios(
    repo: JournalRepository,
    records: list[JournalMetadata],
    pending_updates: bool,
    expected_synchronized: bool,
    expected_missing: int,
    expected_expired: int,
    expected_pending: bool,
) -> None:
    """Verify sync evaluation behaves correctly for combinations of states."""
    repo.config = repo.config.model_copy(update={"journal_update_days": 30})
    repo.records = records
    repo.has_pending_updates = pending_updates

    status = repo.get_sync_status()

    assert status["is_fully_synchronized"] is expected_synchronized
    assert status["missing_metadata_count"] == expected_missing
    assert status["expired_metadata_count"] == expected_expired
    assert status["has_pending_updates"] is expected_pending


def test_merge_new_titles(repo: JournalRepository) -> None:
    """Verify merge of clean input templates avoiding existing overrides."""
    repo.records = [JournalMetadata(input_title="Existing", issn="2049-3630")]
    repo.merge_new_titles(input_titles=["Existing", "Geology", "Geology"])

    assert len(repo) == 2
    assert any(
        r.input_title == "Existing" and r.issn == "2049-3630" for r in repo.records
    )

    new_entries = [r for r in repo.records if r.input_title == "Geology"]
    assert len(new_entries) == 1
    assert new_entries[0].issn is None


def test_update_all_priority_and_limit(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify prioritizing missing metadata and enforcement of capacity limits."""
    caplog.set_level(logging.INFO)
    repo.config = repo.config.model_copy(
        update={"journal_update_limit": 1, "journal_update_days": 30},
    )
    today = datetime.now(UTC).date()
    old_date = str(today - timedelta(days=45))
    recent_date = str(today - timedelta(days=5))

    repo.records = [
        JournalMetadata(
            input_title="Old",
            true_title="Old Journal",
            publisher="Pub",
            issn="0036-8075",
            start_year=2000,
            end_year=2024,
            update=old_date,
        ),
        JournalMetadata(
            input_title="Missing",
            true_title=None,
            publisher=None,
            issn=None,
            start_year=None,
            end_year=None,
            update=recent_date,
        ),
        JournalMetadata(
            input_title="Recent",
            true_title="Recent Journal",
            publisher="Pub",
            issn="0010-051X",
            start_year=2020,
            end_year=2024,
            update=recent_date,
        ),
    ]

    updated_data = [
        JournalMetadata(
            input_title="Missing",
            true_title="Missing Journal",
            publisher="Pub",
            issn="2049-3630",
            start_year=2000,
            end_year=2024,
            update=str(today),
        ),
    ]

    with patch.object(
        JournalRepository,
        "get_journal_metadata",
        return_value=updated_data,
    ) as mock_get:
        repo.update_all()

        assert "Journal categorization completed" in caplog.text
        assert mock_get.call_count == 1
        mock_get.assert_called_with("Missing")
        assert repo.has_pending_updates is True


def test_update_all_sorting_logic_strict(repo: JournalRepository) -> None:
    """Verify completed records are prioritized over templates alphabetically."""
    repo.config = repo.config.model_copy(
        update={"journal_update_limit": 0, "journal_update_days": 30},
    )
    today = datetime.now(UTC).date()
    recent_date = str(today - timedelta(days=5))

    repo.records = [
        JournalMetadata(
            input_title="Alpha Missing",
            true_title=None,
            publisher=None,
            issn=None,
            start_year=None,
            end_year=None,
            update=recent_date,
        ),
        JournalMetadata(
            input_title="Zulu Complete",
            true_title="Zulu Journal",
            publisher="Pub",
            issn="1752-0894",
            start_year=2020,
            end_year=2026,
            update=recent_date,
        ),
        JournalMetadata(
            input_title="Mike Missing",
            true_title=None,
            publisher=None,
            issn=None,
            start_year=None,
            end_year=None,
            update=recent_date,
        ),
        JournalMetadata(
            input_title="Bravo Complete",
            true_title="Bravo Journal",
            publisher="Pub",
            issn="0361-0160",
            start_year=2010,
            end_year=2026,
            update=recent_date,
        ),
    ]

    with patch.object(JournalRepository, "get_journal_metadata", return_value=None):
        repo.update_all()

    assert repo.records[0].input_title == "Bravo Complete"
    assert repo.records[1].input_title == "Zulu Complete"
    assert repo.records[2].input_title == "Alpha Missing"
    assert repo.records[3].input_title == "Mike Missing"


def test_update_batch_empty_new_data_fallback(repo: JournalRepository) -> None:
    """Verify _update_batch retains original record when API returns empty list."""
    record = JournalMetadata(input_title="Empty Lookup J")
    new_records: list[JournalMetadata] = []
    state = UpdateBatchState(total=1)

    repo.config = repo.config.model_copy(update={"journal_update_limit": 1})

    with patch.object(repo, "get_journal_metadata", return_value=[]):
        repo._update_batch([record], new_records, state)

    assert len(new_records) == 1
    assert new_records[0].input_title == "Empty Lookup J"
    assert state.update_count == 1
    assert state.processed == 1

```

These branches are not covered:
- in _filter_matches(), (if raw_title and JournalParser.normalize_title(raw_title) == target_norm) == False
- in update_all(), if self.has_pending_updates == False

---

---

# LLM

```
# tests/unit/repositories/test_journal_repository.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Unit tests for the local journal metadata repository persistence workflows."""

import logging
import time
from datetime import UTC, datetime, timedelta
from typing import Any, Literal
from unittest.mock import MagicMock, patch

import pytest

from citecraft.repositories import JournalRepository
from citecraft.repositories.journal_repository import UpdateBatchState
from citecraft.schemas import JournalMetadata
from citecraft.utils import AppConfig


@pytest.fixture
def repo(test_config: AppConfig) -> JournalRepository:
    """Provide JournalRepository instance utilizing test config."""
    return JournalRepository(config=test_config)


def test_log_heartbeat_if_needed(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify heartbeat logger triggers periodically based on config threshold."""
    repo.config = repo.config.model_copy(
        update={"default_logging_frequency_for_batch_updates": 0.05},
    )
    with caplog.at_level(logging.INFO):
        t1 = time.time() - 0.1
        t2 = repo._log_heartbeat_if_needed(3, 10, t1)
        assert t2 > t1
        assert "Batch update status: 7 updates remaining" in caplog.text

        caplog.clear()
        t3 = repo._log_heartbeat_if_needed(4, 10, t2)
        assert t3 == t2
        assert "Batch update status" not in caplog.text


def test_get_journal_metadata_success(repo: JournalRepository) -> None:
    """Verify successful retrieval of ISSNs and years from endpoints."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {"items": [{"title": "Geology", "ISSN": ["0091-7613"]}]},
    }

    mock_year = MagicMock(status_code=200)
    mock_year.json.return_value = {
        "message": {
            "items": [
                {
                    "published-print": {"date-parts": [[1973]]},
                    "published-online": {"date-parts": [[1995]]},
                },
            ],
        },
    }

    with patch.object(repo.http_client_wrapper, "get") as mock_get:
        mock_get.side_effect = [
            (mock_main, None),
            (mock_year, None),
            (mock_year, None),
        ]

        results = repo.get_journal_metadata("Geology")

        assert len(results) == 1
        assert results[0].issn == "0091-7613"
        assert results[0].start_year == 1973
        assert results[0].end_year == 1995


def test_get_journal_metadata_not_found_behavior(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify fallback to template when no match is found."""
    mock_resp = MagicMock(status_code=200)
    mock_resp.json.return_value = {"message": {"items": []}}

    with (
        patch.object(repo.http_client_wrapper, "get", return_value=(mock_resp, None)),
        caplog.at_level(logging.WARNING),
    ):
        results = repo.get_journal_metadata("Unknown Journal")

        assert "Journal Unknown Journal not found." in caplog.text
        assert len(results) == 1
        assert results[0].issn is None


def test_get_journal_metadata_empty_response(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify fallback when network or API returns a null response."""
    with (
        patch.object(repo.http_client_wrapper, "get", return_value=(None, "mock_url")),
        caplog.at_level(logging.WARNING),
    ):
        results = repo.get_journal_metadata("Some Title")
        assert len(results) == 1
        assert results[0].input_title == "Some Title"
        assert results[0].issn is None
        assert "Empty journal 'Some Title' response from URL" in caplog.text


def test_get_journal_metadata_exact_matches_duplicate_discarded(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify multiple exact matches logs a warning and selects the first match."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {
            "items": [
                {"title": "Duplicate", "ISSN": ["0028-0836"]},
                {"title": "Duplicate", "ISSN": ["8765-4321"]},
            ],
        },
    }
    mock_year = MagicMock(status_code=200)
    mock_year.json.return_value = {
        "message": {"items": [{"published-print": {"date-parts": [[2001]]}}]},
    }

    with (
        patch.object(repo.http_client_wrapper, "get") as mock_get,
        caplog.at_level(logging.WARNING),
    ):
        mock_get.side_effect = [
            (mock_main, None),
            (mock_year, None),
            (mock_year, None),
        ]
        results = repo.get_journal_metadata("Duplicate")
        assert len(results) == 1
        assert results[0].issn == "0028-0836"
        assert "Discarded 2 duplicate titles" in caplog.text


def test_get_journal_metadata_similar_matches_found(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify fuzzy title similarity, filtering, and mapping logic."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {
            "items": [
                {
                    "title": "Nature Geoscience",
                    "publisher": "Springer Nature",
                    "issn": ["1752-0894"],
                },
                {
                    "title": "Nature-Geosciences",
                    "publisher": "Springer Nature",
                    "issn": ["1752-0894", "1752-0908"],
                },
                {
                    "title": "Nature Geoscience",
                    "publisher": "Springer Nature",
                    "issn": ["1752-0894"],
                },
            ],
        },
    }
    mock_year = MagicMock(status_code=200)
    mock_year.json.return_value = {
        "message": {
            "items": [
                {
                    "published-print": {"date-parts": [[2008]]},
                    "published-online": {"date-parts": [[2026]]},
                },
            ],
        },
    }

    with patch.object(repo.http_client_wrapper, "get") as mock_get:
        mock_get.side_effect = [
            (mock_main, None),
            (mock_year, None),
            (mock_year, None),
            (mock_year, None),
            (mock_year, None),
        ]

        with caplog.at_level(logging.WARNING):
            results = repo.get_journal_metadata("nature geoscience")

            assert "Use similar titles to fill metadata" in caplog.text
            assert len(results) == 2
            assert results[0].issn == "1752-0894"
            assert results[1].issn == "1752-0908"


def test_filter_matches_skips_non_matching_and_empty_titles(
    repo: JournalRepository,
) -> None:
    """Verify _filter_matches ignores empty or mismatched titles."""
    items = [
        {"title": ""},  # Triggers empty raw_title branch
        {"title": "Unrelated Title"},  # Triggers unequal normalized title branch
        {"title": "Nature-Geosciences"},  # Triggers match
    ]
    exact_matches, similar_titles = repo._filter_matches(
        "nature geoscience", items
    )
    assert len(exact_matches) == 0
    assert similar_titles == ["Nature-Geosciences"]
    assert len(exact_matches) == 0


def test_get_journal_metadata_missing_issn_handling(repo: JournalRepository) -> None:
    """Verify journals without ISSN are logged and kept without range queries."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {
            "items": [
                {
                    "title": "Natural Hazards",
                    "publisher": "Copernicus Publications",
                },
            ],
        },
    }

    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_main, None),
    ) as mock_get:
        results = repo.get_journal_metadata("Natural Hazards")

        assert len(results) == 1
        assert results[0].issn is None
        assert mock_get.call_count == 1


def test_get_journal_metadata_empty_publication_years(repo: JournalRepository) -> None:
    """Verify default empty year ranges when no published works are detected."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {
            "items": [
                {
                    "title": "Empty Journal",
                    "publisher": "Silent Publisher",
                    "issn": ["0010-051x"],
                },
            ],
        },
    }
    mock_year_empty = MagicMock(status_code=200)
    mock_year_empty.json.return_value = {"message": {"items": []}}

    with patch.object(repo.http_client_wrapper, "get") as mock_get:
        mock_get.side_effect = [
            (mock_main, None),
            (mock_year_empty, None),
            (mock_year_empty, None),
        ]

        results = repo.get_journal_metadata("Empty Journal")

        assert len(results) == 1
        assertresults = [results[0].start_year is None]
        assert results[0].end_year is None
        assert mock_get.call_count == 3


def test_get_issns_by_input_title(repo: JournalRepository) -> None:
    """Verify unicity and extraction of valid ISSNs."""
    repo.records = [
        JournalMetadata(
            input_title="Journal A",
            issn="0010-051x",
            start_year=2000,
            end_year=2010,
        ),
        JournalMetadata(
            input_title="Journal A",
            issn="2049-3630",
            start_year=2005,
            end_year=2015,
        ),
        JournalMetadata(input_title="Journal A", issn=None),
    ]

    assert repo.get_issns_by_input_title("Journal A") == ["0010-051x", "2049-3630"]
    assert repo.get_issns_by_input_title("Non Existent") == []


def test_get_unique_issns_for_titles(repo: JournalRepository) -> None:
    """Verify unique sorting and extraction of ISSNs with normalization rules."""
    repo.records = [
        JournalMetadata(
            input_title="Nature Geoscience",
            issn="1752-0894",
            start_year=2008,
            end_year=2026,
        ),
        JournalMetadata(
            input_title="Journal of Climate",
            issn="2049-3630",
            start_year=2000,
            end_year=2010,
        ),
    ]

    assert repo.get_unique_issns_for_titles([]) == []
    assert repo.get_unique_issns_for_titles(["Unknown"]) == []
    assert repo.get_unique_issns_for_titles(
        ["Nature Geoscience", "Journal of Climate"],
    ) == ["1752-0894", "2049-3630"]


@pytest.mark.parametrize(
    ("order", "mock_items", "expected_year"),
    [
        (
            "asc",
            [
                {
                    "published-print": {"date-parts": [[1990]]},
                    "published-online": {"date-parts": [[2024]]},
                },
            ],
            1990,
        ),
        (
            "desc",
            [
                {
                    "published-print": {"date-parts": [[1990]]},
                    "published-online": {"date-parts": [[2024]]},
                },
            ],
            2024,
        ),
        ("asc", [], None),
        (
            "asc",
            [
                {
                    "published-print": {"date-parts": [[None]]},
                    "published-online": {"date-parts": [[2010]]},
                },
            ],
            2010,
        ),
    ],
)
def test_get_issn_year_endpoint_scenarios(
    repo: JournalRepository,
    order: Literal["asc", "desc"],
    mock_items: list[dict[str, Any]],
    expected_year: int | None,
) -> None:
    """Verify endpoint resolution behaves correctly under different scenarios."""
    mock_response = MagicMock()
    mock_response.json.return_value = {"message": {"items": mock_items}}

    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_response, None),
    ) as mock_get:
        assert repo.get_issn_year_endpoint("2049-3630", order) == expected_year
        mock_get.assert_called_once()


def test_get_issn_year_endpoint_empty_response(repo: JournalRepository) -> None:
    """Verify endpoint recovery when response resolves to None."""
    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(None, "mock_url"),
    ) as mock_get:
        assert repo.get_issn_year_endpoint("1234-5678", "asc") is None
        mock_get.assert_called_once()


def test_get_issn_year_endpoint_no_valid_years(repo: JournalRepository) -> None:
    """Verify endpoint resolution when payload exists but contains only null dates."""
    mock_response = MagicMock()
    mock_response.json.return_value = {
        "message": {
            "items": [
                {
                    "published-print": {"date-parts": [[None]]},
                    "published-online": {"date-parts": [[None]]},
                },
            ],
        },
    }
    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_response, None),
    ):
        assert repo.get_issn_year_endpoint("2049-3630", "asc") is None


@pytest.mark.parametrize(
    (
        "records",
        "pending_updates",
        "expected_synchronized",
        "expected_missing",
        "expected_expired",
        "expected_pending",
    ),
    [
        (
            [
                JournalMetadata(
                    input_title="Valid J",
                    true_title="T",
                    publisher="P",
                    issn="2049-3630",
                    start_year=2000,
                    end_year=2026,
                    update=str(
                        datetime.now(UTC).date() - timedelta(days=5),
                    ),
                ),
                JournalMetadata(
                    input_title="Expired J",
                    true_title="T",
                    publisher="P",
                    issn="1752-0908",
                    start_year=2000,
                    end_year=2026,
                    update=str(datetime.now(UTC).date() - timedelta(days=45)),
                ),
                JournalMetadata(
                    input_title="Missing J",
                    true_title=None,
                    publisher=None,
                    issn=None,
                    start_year=None,
                    end_year=None,
                    update=str(datetime.now(UTC).date() - timedelta(days=5)),
                ),
            ],
            True,
            False,
            1,
            1,
            True,
        ),
        (
            [
                JournalMetadata(
                    input_title="Valid J",
                    true_title="T",
                    publisher="P",
                    issn="2049-3630",
                    start_year=2000,
                    end_year=2026,
                    update=str(datetime.now(UTC).date() - timedelta(days=5)),
                ),
            ],
            False,
            True,
            0,
            0,
            False,
        ),
    ],
)
def test_get_sync_status_scenarios(
    repo: JournalRepository,
    records: list[JournalMetadata],
    pending_updates: bool,
    expected_synchronized: bool,
    expected_missing: int,
    expected_expired: int,
    expected_pending: bool,
) -> None:
    """Verify sync evaluation behaves correctly for combinations of states."""
    repo.config = repo.config.model_copy(update={"journal_update_days": 30})
    repo.records = records
    repo.has_pending_updates = pending_updates

    status = repo.get_sync_status()

    assert status["is_fully_synchronized"] is expected_synchronized
    assert status["missing_metadata_count"] == expected_missing
    assert status["expired_metadata_count"] == expected_expired
    assert status["has_pending_updates"] is expected_pending


def test_merge_new_titles(repo: JournalRepository) -> None:
    """Verify merge of clean input templates avoiding existing overrides."""
    repo.records = [JournalMetadata(input_title="Existing", issn="2049-3630")]
    repo.merge_new_titles(input_titles=["Existing", "Geology", "Geology"])

    assert len(repo) == 2
    assert any(
        r.input_title == "Existing" and r.issn == "2049-3630" for r in repo.records
    )

    new_entries = [r for r in repo.records if r.input_title == "Geology"]
    assert len(new_entries) == 1
    assert new_entries[0].issn is None


def test_update_all_priority_and_limit(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify prioritizing missing metadata and enforcement of capacity limits."""
    caplog.set_level(logging.INFO)
    repo.config = repo.config.model_copy(
        update={"journal_update_limit": 1, "journal_update_days": 30},
    )
    today = datetime.now(UTC).date()
    old_date = str(today - timedelta(days=45))
    recent_date = str(today - timedelta(days=5))

    repo.records = [
        JournalMetadata(
            input_title="Old",
            true_title="Old Journal",
            publisher="Pub",
            issn="0036-8075",
            start_year=2000,
            end_year=2024,
            update=old_date,
        ),
        JournalMetadata(
            input_title="Missing",
            true_title=None,
            publisher=None,
            issn=None,
            start_year=None,
            end_year=None,
            update=recent_date,
        ),
        JournalMetadata(
            input_title="Recent",
            true_title="Recent Journal",
            publisher="Pub",
            issn="0010-051X",
            start_year=2020,
            end_year=2024,
            update=recent_date,
        ),
    ]

    updated_data = [
        JournalMetadata(
            input_title="Missing",
            true_title="Missing Journal",
            publisher="Pub",
            issn="2049-3630",
            start_year=2000,
            end_year=2024,
            update=str(today),
        ),
    ]

    with patch.object(
        JournalRepository,
        "get_journal_metadata",
        return_value=updated_data,
    ) as mock_get:
        repo.update_all()

        assert "Journal categorization completed" in caplog.text
        assert mock_get.call_count == 1
        mock_get.assert_called_with("Missing")
        assert repo.has_pending_updates is True


def test_update_all_no_pending_updates_under_limit(
    repo: JournalRepository,
) -> None:
    """Verify update_all clears has_pending_updates when limit is not exceeded."""
    repo.config = repo.config.model_copy(
        update={"journal_update_limit": 10, "journal_update_days": 30},
    )
    today = datetime.now(UTC).date()
    old_date = str(today - timedelta(days=45))

    repo.records = [
        JournalMetadata(
            input_title="Expired",
            true_title="Expired J",
            publisher="Pub",
            issn="0036-8075",
            start_year=2000,
            end_year=2024,
            update=old_date,
        ),
    ]

    updated_data = [
        JournalMetadata(
            input_title="Expired",
            true_title="Expired J",
            publisher="Pub",
            issn="0036-8075",
            start_year=2000,
            end_year=2024,
            update=str(today),
        ),
    ]

    with patch.object(
        JournalRepository,
        "get_journal_metadata",
        return_value=updated_data,
    ):
        repo.update_all()
        assert repo.has_pending_updates is False


def test_update_all_sorting_logic_strict(repo: JournalRepository) -> None:
    """Verify completed records are prioritized over templates alphabetically."""
    repo.config = repo.config.model_copy(
        update={"journal_update_limit": 0, "journal_update_days": 30},
    )
    today = datetime.now(UTC).date()
    recent_date = str(today - timedelta(days=5))

    repo.records = [
        JournalMetadata(
            input_title="Alpha Missing",
            true_title=None,
            publisher=None,
            issn=None,
            start_year=None,
            end_year=None,
            update=recent_date,
        ),
        JournalMetadata(
            input_title="Zulu Complete",
            true_title="Zulu Journal",
            publisher="Pub",
            issn="1752-0894",
            start_year=2020,
            end_year=2026,
            update=recent_date,
        ),
        JournalMetadata(
            input_title="Mike Missing",
            true_title=None,
            publisher=None,
            issn=None,
            start_year=None,
            end_year=None,
            update=recent_date,
        ),
        JournalMetadata(
            input_title="Bravo Complete",
            true_title="Bravo Journal",
            publisher="Pub",
            issn="0361-0160",
            start_year=2010,
            end_year=2026,
            update=recent_date,
        ),
    ]

    with patch.object(JournalRepository, "get_journal_metadata", return_value=None):
        repo.update_all()

    assert repo.records[0].input_title == "Bravo Complete"
    assert repo.records[1].input_title == "Zulu Complete"
    assert repo.records[2].input_title == "Alpha Missing"
    assert repo.records[3].input_title == "Mike Missing"


def test_update_batch_empty_new_data_fallback(repo: JournalRepository) -> None:
    """Verify _update_batch retains original record when API returns empty list."""
    record = JournalMetadata(input_title="Empty Lookup J")
    new_records: list[JournalMetadata] = []
    state = UpdateBatchState(total=1)

    repo.config = repo.config.model_copy(update={"journal_update_limit": 1})

    with patch.object(repo, "get_journal_metadata", return_value=[]):
        repo._update_batch([record], new_records, state)

    assert len(new_records) == 1
    assert new_records[0].input_title == "Empty Lookup J"
    assert state.update_count == 1
    assert state.processed == 1
```

---

---

# ME

I have this error:
```
test_filter_matches_skips_non_matching_and_empty_titles _____________________________________________
tests\unit\repositories\test_journal_repository.py:219: in test_filter_matches_skips_non_matching_and_empty_titles
    assert len(exact_matches) == 0
E   AssertionError: assert 1 == 0
E    +  where 1 = len([{'title': 'Nature-Geosciences'}])
---------------------------------------------------------------- Captured log call ----------------------------------------------------------------
WARNING  citecraft.repositories.journal_repository:journal_repository.py:176 Journal nature geoscience not found. Use similar titles to fill metadata: Nature-Geosciences
```

---

---

# LLM

```
# tests/unit/repositories/test_journal_repository.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Unit tests for the local journal metadata repository persistence workflows."""

import logging
import time
from datetime import UTC, datetime, timedelta
from typing import Any, Literal
from unittest.mock import MagicMock, patch

import pytest

from citecraft.repositories import JournalRepository
from citecraft.repositories.journal_repository import UpdateBatchState
from citecraft.schemas import JournalMetadata
from citecraft.utils import AppConfig


@pytest.fixture
def repo(test_config: AppConfig) -> JournalRepository:
    """Provide JournalRepository instance utilizing test config."""
    return JournalRepository(config=test_config)


def test_log_heartbeat_if_needed(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify heartbeat logger triggers periodically based on config threshold."""
    repo.config = repo.config.model_copy(
        update={"default_logging_frequency_for_batch_updates": 0.05},
    )
    with caplog.at_level(logging.INFO):
        t1 = time.time() - 0.1
        t2 = repo._log_heartbeat_if_needed(3, 10, t1)
        assert t2 > t1
        assert "Batch update status: 7 updates remaining" in caplog.text

        caplog.clear()
        t3 = repo._log_heartbeat_if_needed(4, 10, t2)
        assert t3 == t2
        assert "Batch update status" not in caplog.text


def test_get_journal_metadata_success(repo: JournalRepository) -> None:
    """Verify successful retrieval of ISSNs and years from endpoints."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {"items": [{"title": "Geology", "ISSN": ["0091-7613"]}]},
    }

    mock_year = MagicMock(status_code=200)
    mock_year.json.return_value = {
        "message": {
            "items": [
                {
                    "published-print": {"date-parts": [[1973]]},
                    "published-online": {"date-parts": [[1995]]},
                },
            ],
        },
    }

    with patch.object(repo.http_client_wrapper, "get") as mock_get:
        mock_get.side_effect = [
            (mock_main, None),
            (mock_year, None),
            (mock_year, None),
        ]

        results = repo.get_journal_metadata("Geology")

        assert len(results) == 1
        assert results[0].issn == "0091-7613"
        assert results[0].start_year == 1973
        assert results[0].end_year == 1995


def test_get_journal_metadata_not_found_behavior(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify fallback to template when no match is found."""
    mock_resp = MagicMock(status_code=200)
    mock_resp.json.return_value = {"message": {"items": []}}

    with (
        patch.object(repo.http_client_wrapper, "get", return_value=(mock_resp, None)),
        caplog.at_level(logging.WARNING),
    ):
        results = repo.get_journal_metadata("Unknown Journal")

        assert "Journal Unknown Journal not found." in caplog.text
        assert len(results) == 1
        assert results[0].issn is None


def test_get_journal_metadata_empty_response(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify fallback when network or API returns a null response."""
    with (
        patch.object(repo.http_client_wrapper, "get", return_value=(None, "mock_url")),
        caplog.at_level(logging.WARNING),
    ):
        results = repo.get_journal_metadata("Some Title")
        assert len(results) == 1
        assert results[0].input_title == "Some Title"
        assert results[0].issn is None
        assert "Empty journal 'Some Title' response from URL" in caplog.text


def test_get_journal_metadata_exact_matches_duplicate_discarded(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify multiple exact matches logs a warning and selects the first match."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {
            "items": [
                {"title": "Duplicate", "ISSN": ["0028-0836"]},
                {"title": "Duplicate", "ISSN": ["8765-4321"]},
            ],
        },
    }
    mock_year = MagicMock(status_code=200)
    mock_year.json.return_value = {
        "message": {"items": [{"published-print": {"date-parts": [[2001]]}}]},
    }

    with (
        patch.object(repo.http_client_wrapper, "get") as mock_get,
        caplog.at_level(logging.WARNING),
    ):
        mock_get.side_effect = [
            (mock_main, None),
            (mock_year, None),
            (mock_year, None),
        ]
        results = repo.get_journal_metadata("Duplicate")
        assert len(results) == 1
        assert results[0].issn == "0028-0836"
        assert "Discarded 2 duplicate titles" in caplog.text


def test_get_journal_metadata_similar_matches_found(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify fuzzy title similarity, filtering, and mapping logic."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {
            "items": [
                {
                    "title": "Nature Geoscience",
                    "publisher": "Springer Nature",
                    "issn": ["1752-0894"],
                },
                {
                    "title": "Nature-Geosciences",
                    "publisher": "Springer Nature",
                    "issn": ["1752-0894", "1752-0908"],
                },
                {
                    "title": "Nature Geoscience",
                    "publisher": "Springer Nature",
                    "issn": ["1752-0894"],
                },
            ],
        },
    }
    mock_year = MagicMock(status_code=200)
    mock_year.json.return_value = {
        "message": {
            "items": [
                {
                    "published-print": {"date-parts": [[2008]]},
                    "published-online": {"date-parts": [[2026]]},
                },
            ],
        },
    }

    with patch.object(repo.http_client_wrapper, "get") as mock_get:
        mock_get.side_effect = [
            (mock_main, None),
            (mock_year, None),
            (mock_year, None),
            (mock_year, None),
            (mock_year, None),
        ]

        with caplog.at_level(logging.WARNING):
            results = repo.get_journal_metadata("nature geoscience")

            assert "Use similar titles to fill metadata" in caplog.text
            assert len(results) == 2
            assert results[0].issn == "1752-0894"
            assert results[1].issn == "1752-0908"


def test_filter_matches_skips_non_matching_and_empty_titles(
    repo: JournalRepository,
) -> None:
    """Verify _filter_matches ignores empty or mismatched titles."""
    items = [
        {"title": ""},  # Triggers empty raw_title branch
        {"title": "Unrelated Title"},  # Triggers unequal normalized title branch
        {"title": "Nature-Geosciences"},  # Triggers match
    ]
    matches, matched_titles = repo._filter_matches(
        "nature geoscience", items
    )
    assert len(matches) == 1
    assert matches[0]["title"] == "Nature-Geosciences"
    assert matched_titles == ["Nature-Geosciences"]


def test_get_journal_metadata_missing_issn_handling(repo: JournalRepository) -> None:
    """Verify journals without ISSN are logged and kept without range queries."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {
            "items": [
                {
                    "title": "Natural Hazards",
                    "publisher": "Copernicus Publications",
                },
            ],
        },
    }

    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_main, None),
    ) as mock_get:
        results = repo.get_journal_metadata("Natural Hazards")

        assert len(results) == 1
        assert results[0].issn is None
        assert mock_get.call_count == 1


def test_get_journal_metadata_empty_publication_years(repo: JournalRepository) -> None:
    """Verify default empty year ranges when no published works are detected."""
    mock_main = MagicMock(status_code=200)
    mock_main.json.return_value = {
        "message": {
            "items": [
                {
                    "title": "Empty Journal",
                    "publisher": "Silent Publisher",
                    "issn": ["0010-051x"],
                },
            ],
        },
    }
    mock_year_empty = MagicMock(status_code=200)
    mock_year_empty.json.return_value = {"message": {"items": []}}

    with patch.object(repo.http_client_wrapper, "get") as mock_get:
        mock_get.side_effect = [
            (mock_main, None),
            (mock_year_empty, None),
            (mock_year_empty, None),
        ]

        results = repo.get_journal_metadata("Empty Journal")

        assert len(results) == 1
        assert results[0].start_year is None
        assert results[0].end_year is None
        assert mock_get.call_count == 3


def test_get_issns_by_input_title(repo: JournalRepository) -> None:
    """Verify unicity and extraction of valid ISSNs."""
    repo.records = [
        JournalMetadata(
            input_title="Journal A",
            issn="0010-051x",
            start_year=2000,
            end_year=2010,
        ),
        JournalMetadata(
            input_title="Journal A",
            issn="2049-3630",
            start_year=2005,
            end_year=2015,
        ),
        JournalMetadata(input_title="Journal A", issn=None),
    ]

    assert repo.get_issns_by_input_title("Journal A") == ["0010-051x", "2049-3630"]
    assert repo.get_issns_by_input_title("Non Existent") == []


def test_get_unique_issns_for_titles(repo: JournalRepository) -> None:
    """Verify unique sorting and extraction of ISSNs with normalization rules."""
    repo.records = [
        JournalMetadata(
            input_title="Nature Geoscience",
            issn="1752-0894",
            start_year=2008,
            end_year=2026,
        ),
        JournalMetadata(
            input_title="Journal of Climate",
            issn="2049-3630",
            start_year=2000,
            end_year=2010,
        ),
    ]

    assert repo.get_unique_issns_for_titles([]) == []
    assert repo.get_unique_issns_for_titles(["Unknown"]) == []
    assert repo.get_unique_issns_for_titles(
        ["Nature Geoscience", "Journal of Climate"],
    ) == ["1752-0894", "2049-3630"]


@pytest.mark.parametrize(
    ("order", "mock_items", "expected_year"),
    [
        (
            "asc",
            [
                {
                    "published-print": {"date-parts": [[1990]]},
                    "published-online": {"date-parts": [[2024]]},
                },
            ],
            1990,
        ),
        (
            "desc",
            [
                {
                    "published-print": {"date-parts": [[1990]]},
                    "published-online": {"date-parts": [[2024]]},
                },
            ],
            2024,
        ),
        ("asc", [], None),
        (
            "asc",
            [
                {
                    "published-print": {"date-parts": [[None]]},
                    "published-online": {"date-parts": [[2010]]},
                },
            ],
            2010,
        ),
    ],
)
def test_get_issn_year_endpoint_scenarios(
    repo: JournalRepository,
    order: Literal["asc", "desc"],
    mock_items: list[dict[str, Any]],
    expected_year: int | None,
) -> None:
    """Verify endpoint resolution behaves correctly under different scenarios."""
    mock_response = MagicMock()
    mock_response.json.return_value = {"message": {"items": mock_items}}

    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_response, None),
    ) as mock_get:
        assert repo.get_issn_year_endpoint("2049-3630", order) == expected_year
        mock_get.assert_called_once()


def test_get_issn_year_endpoint_empty_response(repo: JournalRepository) -> None:
    """Verify endpoint recovery when response resolves to None."""
    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(None, "mock_url"),
    ) as mock_get:
        assert repo.get_issn_year_endpoint("1234-5678", "asc") is None
        mock_get.assert_called_once()


def test_get_issn_year_endpoint_no_valid_years(repo: JournalRepository) -> None:
    """Verify endpoint resolution when payload exists but contains only null dates."""
    mock_response = MagicMock()
    mock_response.json.return_value = {
        "message": {
            "items": [
                {
                    "published-print": {"date-parts": [[None]]},
                    "published-online": {"date-parts": [[None]]},
                },
            ],
        },
    }
    with patch.object(
        repo.http_client_wrapper,
        "get",
        return_value=(mock_response, None),
    ):
        assert repo.get_issn_year_endpoint("2049-3630", "asc") is None


@pytest.mark.parametrize(
    (
        "records",
        "pending_updates",
        "expected_synchronized",
        "expected_missing",
        "expected_expired",
        "expected_pending",
    ),
    [
        (
            [
                JournalMetadata(
                    input_title="Valid J",
                    true_title="T",
                    publisher="P",
                    issn="2049-3630",
                    start_year=2000,
                    end_year=2026,
                    update=str(
                        datetime.now(UTC).date() - timedelta(days=5),
                    ),
                ),
                JournalMetadata(
                    input_title="Expired J",
                    true_title="T",
                    publisher="P",
                    issn="1752-0908",
                    start_year=2000,
                    end_year=2026,
                    update=str(datetime.now(UTC).date() - timedelta(days=45)),
                ),
                JournalMetadata(
                    input_title="Missing J",
                    true_title=None,
                    publisher=None,
                    issn=None,
                    start_year=None,
                    end_year=None,
                    update=str(datetime.now(UTC).date() - timedelta(days=5)),
                ),
            ],
            True,
            False,
            1,
            1,
            True,
        ),
        (
            [
                JournalMetadata(
                    input_title="Valid J",
                    true_title="T",
                    publisher="P",
                    issn="2049-3630",
                    start_year=2000,
                    end_year=2026,
                    update=str(datetime.now(UTC).date() - timedelta(days=5)),
                ),
            ],
            False,
            True,
            0,
            0,
            False,
        ),
    ],
)
def test_get_sync_status_scenarios(
    repo: JournalRepository,
    records: list[JournalMetadata],
    pending_updates: bool,
    expected_synchronized: bool,
    expected_missing: int,
    expected_expired: int,
    expected_pending: bool,
) -> None:
    """Verify sync evaluation behaves correctly for combinations of states."""
    repo.config = repo.config.model_copy(update={"journal_update_days": 30})
    repo.records = records
    repo.has_pending_updates = pending_updates

    status = repo.get_sync_status()

    assert status["is_fully_synchronized"] is expected_synchronized
    assert status["missing_metadata_count"] == expected_missing
    assert status["expired_metadata_count"] == expected_expired
    assert status["has_pending_updates"] is expected_pending


def test_merge_new_titles(repo: JournalRepository) -> None:
    """Verify merge of clean input templates avoiding existing overrides."""
    repo.records = [JournalMetadata(input_title="Existing", issn="2049-3630")]
    repo.merge_new_titles(input_titles=["Existing", "Geology", "Geology"])

    assert len(repo) == 2
    assert any(
        r.input_title == "Existing" and r.issn == "2049-3630" for r in repo.records
    )

    new_entries = [r for r in repo.records if r.input_title == "Geology"]
    assert len(new_entries) == 1
    assert new_entries[0].issn is None


def test_update_all_priority_and_limit(
    repo: JournalRepository,
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify prioritizing missing metadata and enforcement of capacity limits."""
    caplog.set_level(logging.INFO)
    repo.config = repo.config.model_copy(
        update={"journal_update_limit": 1, "journal_update_days": 30},
    )
    today = datetime.now(UTC).date()
    old_date = str(today - timedelta(days=45))
    recent_date = str(today - timedelta(days=5))

    repo.records = [
        JournalMetadata(
            input_title="Old",
            true_title="Old Journal",
            publisher="Pub",
            issn="0036-8075",
            start_year=2000,
            end_year=2024,
            update=old_date,
        ),
        JournalMetadata(
            input_title="Missing",
            true_title=None,
            publisher=None,
            issn=None,
            start_year=None,
            end_year=None,
            update=recent_date,
        ),
        JournalMetadata(
            input_title="Recent",
            true_title="Recent Journal",
            publisher="Pub",
            issn="0010-051X",
            start_year=2020,
            end_year=2024,
            update=recent_date,
        ),
    ]

    updated_data = [
        JournalMetadata(
            input_title="Missing",
            true_title="Missing Journal",
            publisher="Pub",
            issn="2049-3630",
            start_year=2000,
            end_year=2024,
            update=str(today),
        ),
    ]

    with patch.object(
        JournalRepository,
        "get_journal_metadata",
        return_value=updated_data,
    ) as mock_get:
        repo.update_all()

        assert "Journal categorization completed" in caplog.text
        assert mock_get.call_count == 1
        mock_get.assert_called_with("Missing")
        assert repo.has_pending_updates is True


def test_update_all_no_pending_updates_under_limit(
    repo: JournalRepository,
) -> None:
    """Verify update_all clears has_pending_updates when limit is not exceeded."""
    repo.config = repo.config.model_copy(
        update={"journal_update_limit": 10, "journal_update_days": 30},
    )
    today = datetime.now(UTC).date()
    old_date = str(today - timedelta(days=45))

    repo.records = [
        JournalMetadata(
            input_title="Expired",
            true_title="Expired J",
            publisher="Pub",
            issn="0036-8075",
            start_year=2000,
            end_year=2024,
            update=old_date,
        ),
    ]

    updated_data = [
        JournalMetadata(
            input_title="Expired",
            true_title="Expired J",
            publisher="Pub",
            issn="0036-8075",
            start_year=2000,
            end_year=2024,
            update=str(today),
        ),
    ]

    with patch.object(
        JournalRepository,
        "get_journal_metadata",
        return_value=updated_data,
    ):
        repo.update_all()
        assert repo.has_pending_updates is False


def test_update_all_sorting_logic_strict(repo: JournalRepository) -> None:
    """Verify completed records are prioritized over templates alphabetically."""
    repo.config = repo.config.model_copy(
        update={"journal_update_limit": 0, "journal_update_days": 30},
    )
    today = datetime.now(UTC).date()
    recent_date = str(today - timedelta(days=5))

    repo.records = [
        JournalMetadata(
            input_title="Alpha Missing",
            true_title=None,
            publisher=None,
            issn=None,
            start_year=None,
            end_year=None,
            update=recent_date,
        ),
        JournalMetadata(
            input_title="Zulu Complete",
            true_title="Zulu Journal",
            publisher="Pub",
            issn="1752-0894",
            start_year=2020,
            end_year=2026,
            update=recent_date,
        ),
        JournalMetadata(
            input_title="Mike Missing",
            true_title=None,
            publisher=None,
            issn=None,
            start_year=None,
            end_year=None,
            update=recent_date,
        ),
        JournalMetadata(
            input_title="Bravo Complete",
            true_title="Bravo Journal",
            publisher="Pub",
            issn="0361-0160",
            start_year=2010,
            end_year=2026,
            update=recent_date,
        ),
    ]

    with patch.object(JournalRepository, "get_journal_metadata", return_value=None):
        repo.update_all()

    assert repo.records[0].input_title == "Bravo Complete"
    assert repo.records[1].input_title == "Zulu Complete"
    assert repo.records[2].input_title == "Alpha Missing"
    assert repo.records[3].input_title == "Mike Missing"


def test_update_batch_empty_new_data_fallback(repo: JournalRepository) -> None:
    """Verify _update_batch retains original record when API returns empty list."""
    record = JournalMetadata(input_title="Empty Lookup J")
    new_records: list[JournalMetadata] = []
    state = UpdateBatchState(total=1)

    repo.config = repo.config.model_copy(update={"journal_update_limit": 1})

    with patch.object(repo, "get_journal_metadata", return_value=[]):
        repo._update_batch([record], new_records, state)

    assert len(new_records) == 1
    assert new_records[0].input_title == "Empty Lookup J"
    assert state.update_count == 1
    assert state.processed == 1
```

---

---

# ME

```
# src/citecraft/adapters/citeproc_adapter.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Adapter for interfacing with the citeproc-py bibliography engine."""

import io
import logging
import warnings
from typing import Any

import citeproc.formatter
from citeproc import (
    Citation,
    CitationItem,
    CitationStylesBibliography,
    CitationStylesStyle,
)
from citeproc.source.json import CiteProcJSON

logger = logging.getLogger(__name__)


class CiteprocAdapter:
    """Encapsulates citeproc-py use, filtering logs and standardizing failures."""

    @staticmethod
    def create_json_source(
        csl_dict: dict[str, Any],
        *,
        doi: str,
    ) -> tuple[CiteProcJSON | None, str | None]:
        """Instantiate CiteProcJSON source and handle unsupported field warnings."""
        try:
            with warnings.catch_warnings(record=True) as captured_warnings:
                warnings.simplefilter("always", UserWarning)
                source = CiteProcJSON([csl_dict])
                for w in captured_warnings:
                    msg = str(w.message)
                    if "unsupported" in msg:
                        logger.debug(
                            (
                                "Citeproc-py stripped unsupported fields for DOI: %s. "
                                "Details: %s"
                            ),
                            doi,
                            msg,
                            extra={
                                "status": "WARN",
                                "event": "citeproc_unsupported_fields_filtered",
                                "doi": doi,
                                "citeproc_message": msg,
                            },
                        )
                    else:
                        logger.warning(
                            "Unexpected citeproc-py warning for DOI: %s. Details: %s",
                            doi,
                            msg,
                            extra={
                                "status": "WARN",
                                "event": "citeproc_unexpected_warning_captured",
                                "doi": doi,
                                "warning_category": w.category.__name__
                                if w.category
                                else "Unknown",
                            },
                        )
                return source, None

        except (TypeError, ValueError, KeyError, UnboundLocalError) as e:
            error_msg = str(e)
            hint = ""
            if isinstance(e, UnboundLocalError):
                hint = (
                    " (Check presence/validity of type/DOI fields or potential "
                    "citeproc-py package update issue)"
                )

            full_error_context = f"{error_msg}{hint}" if error_msg else hint.strip()

            logger.warning(
                "CiteProcJSON structural translation failed for DOI: %s. Reason: %s",
                doi,
                full_error_context,
                extra={
                    "status": "KO",
                    "event": "citeproc_source_generation_failed",
                    "doi": doi,
                },
            )
            return None, full_error_context

    @staticmethod
    def parse_csl_style(
        style_content: str,
        *,
        doi: str,
    ) -> tuple[CitationStylesStyle | None, str | None]:
        """Parse XML CSL style sheet definitions safely without external XSD checks."""
        try:
            style_bytes = style_content.encode("utf-8")
            style_file = io.BytesIO(style_bytes)
            bib_style = CitationStylesStyle(style_file, validate=False)

        except Exception as e:
            error_msg = str(e)
            logger.warning(
                "Failed to parse CSL style definitions for DOI: %s. Details: %s",
                doi,
                error_msg,
                extra={
                    "status": "KO",
                    "event": "citeproc_style_parsing_failed",
                    "doi": doi,
                },
            )
            return None, error_msg
        else:
            return bib_style, None

    @staticmethod
    def render_bibliography(
        bib_style: CitationStylesStyle,
        bib_source: CiteProcJSON,
        *,
        item_id: str,
        doi: str,
    ) -> tuple[str | None, str | None]:
        """Produce final text bibliography strings."""
        try:
            bibliography = CitationStylesBibliography(
                bib_style,
                bib_source,
                citeproc.formatter.plain,
            )
            citation = Citation([CitationItem(item_id)])
            bibliography.register(citation)

            # Execution triggers internal rendering pipeline
            render_outputs = bibliography.bibliography()
            if render_outputs and len(render_outputs) > 0:
                rendered_text = str(render_outputs[0])
                logger.debug(
                    (
                        "Successfully resolved bibliography reference generation for "
                        "DOI: %s"
                    ),
                    doi,
                    extra={
                        "status": "OK",
                        "event": "doi_local_resolution_success",
                        "doi": doi,
                    },
                )
                return rendered_text, None

        except AttributeError as e:
            if "'NoneType' object has no attribute 'render'" in str(e):
                error_msg = (
                    "The CSL style file hosted on GitHub appears to be corrupted "
                    "or structurally incompatible with citeproc-py."
                )
                logger.warning(
                    "CSL parsing failure for DOI: %s. Technical detail: %s",
                    doi,
                    error_msg,
                    extra={
                        "status": "KO",
                        "event": "citeproc_github_style_file_corrupted",
                        "doi": doi,
                        "exception_details": str(e),
                    },
                )
                return None, error_msg

            # Re-raise if it's an unrelated AttributeError
            raise

        except Exception as e:
            error_msg = str(e)
            logger.warning(
                (
                    "Bibliography rendering pipeline crashed for DOI: %s. "
                    "Technical error: %s. Please create a github issue if recurrent."
                ),
                doi,
                error_msg,
                extra={
                    "status": "KO",
                    "event": "citeproc_rendering_pipeline_crashed",
                    "doi": doi,
                },
            )
            return None, error_msg
        else:
            return None, "Empty layout produced by bibliography generator."

# tests/unit/adapters/test_citeproc_adapter.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Unit tests for the citeproc adapter layer."""

import logging
from pathlib import Path
from typing import Any
from unittest.mock import patch

import pytest

from citecraft.adapters import CiteprocAdapter

# Dynamically load a minimalist valid XML CSL style mock derived from nature.csl
FIXTURE_DIR = Path(__file__).parent / "fixtures"
MOCK_CSL_STYLE = (FIXTURE_DIR / "mock_csl.xml").read_text(encoding="utf-8")


def test_create_json_source_success_with_unsupported_warning(
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify instantiation redirects citeproc internal warnings onto local logs."""
    csl_dict = {
        "id": "10.1000/xyz123",
        "type": "article-journal",
        "title": "Test Title",
        "published": "2026-01-01",  # Triggers citeproc unsupported variable warning
    }

    with caplog.at_level(logging.DEBUG):
        source, err = CiteprocAdapter.create_json_source(csl_dict, doi="10.1000/xyz123")

    assert source is not None
    assert err is None

    # Check warning redirection to debug log channel
    debug_records = [r for r in caplog.records if r.levelname == "DEBUG"]
    assert len(debug_records) == 1
    assert "citeproc_unsupported_fields_filtered" in getattr(
        debug_records[0],
        "event",
        "",
    )


@pytest.mark.parametrize(
    "broken_csl_payload",
    [
        {"id": "10.1000/xyz123"},  # Missing entirely the required 'type'
        {"type": "article-journal"},  # Missing entirely the required 'id'
    ],
)
# WARNING: we don't the fields set to None, citeproc-py has another behavior that is
# inconsistent
def test_create_json_source_malformed_payloads(
    broken_csl_payload: dict[str, Any],
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify structural type and ID anomalies handle standard citeproc failures."""
    with caplog.at_level(logging.WARNING):
        source, err = CiteprocAdapter.create_json_source(
            broken_csl_payload,
            doi="10.1000/xyz123",
        )

    assert source is None
    assert err is not None
    assert len(caplog.records) == 1
    assert caplog.records[0].levelname == "WARNING"


def test_parse_csl_style_success() -> None:
    """Verify successful parsing logic transformation onto native citeproc elements."""
    style, err = CiteprocAdapter.parse_csl_style(MOCK_CSL_STYLE, doi="10.1000/xyz123")
    assert style is not None
    assert err is None


def test_parse_csl_style_failure(caplog: pytest.LogCaptureFixture) -> None:
    """Verify broken layout XML code captures errors."""
    malformed_xml = "<style><invalid-unclosed-tag>"

    with caplog.at_level(logging.WARNING):
        style, err = CiteprocAdapter.parse_csl_style(
            malformed_xml,
            doi="10.1000/xyz123",
        )

    assert style is None
    assert err is not None
    assert len(caplog.records) == 1


def test_full_rendering_pipeline_success(caplog: pytest.LogCaptureFixture) -> None:
    """Verify end-to-end processing sequence through the adapter layers."""
    doi_key = "10.1000/xyz123"
    csl_dict = {
        "id": doi_key,
        "type": "article-journal",
        "title": "Glacier Melting Dynamics",
    }
    expected_test = (
        "1.Glacier Melting Dynamics.."  # WARNING Dependent on MOCK_CSL_STYLE
    )

    source, _ = CiteprocAdapter.create_json_source(csl_dict, doi=doi_key)
    style, _ = CiteprocAdapter.parse_csl_style(MOCK_CSL_STYLE, doi=doi_key)

    assert source is not None
    assert style is not None

    with caplog.at_level(logging.DEBUG):
        rendered_text, err = CiteprocAdapter.render_bibliography(
            style,
            source,
            item_id=doi_key,
            doi=doi_key,
        )

    assert err is None
    assert rendered_text == expected_test

    # Confirm final resolution tracing assertion
    ok_records = [
        r
        for r in caplog.records
        if r.levelname == "DEBUG"
        and "doi_local_resolution_success" in getattr(r, "event", "")
    ]
    assert len(ok_records) == 1


def test_render_bibliography_corrupted_github_style_handles_attribute_error(
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify that an AttributeError triggers a warning and fallback."""
    doi_key = "10.1000/xyz123"
    csl_dict = {
        "id": doi_key,
        "type": "article-journal",
        "title": "Glacier Melting Dynamics",
    }

    source, _ = CiteprocAdapter.create_json_source(csl_dict, doi=doi_key)
    style, _ = CiteprocAdapter.parse_csl_style(MOCK_CSL_STYLE, doi=doi_key)

    assert source is not None
    assert style is not None

    with (
        patch(
            "citeproc.CitationStylesBibliography.bibliography",
            side_effect=AttributeError("'NoneType' object has no attribute 'render'"),
        ),
        caplog.at_level(logging.WARNING),
    ):
        rendered_text, err = CiteprocAdapter.render_bibliography(
            style,
            source,
            item_id=doi_key,
            doi=doi_key,
        )

    assert rendered_text is None
    assert err is not None
    assert "hosted on GitHub appears to be corrupted" in err
    assert len(caplog.records) == 1
    record = caplog.records[0]
    assert record.levelname == "WARNING"
    assert getattr(record, "event", "") == "citeproc_github_style_file_corrupted"
    assert getattr(record, "doi", "") == doi_key
```

These branches are not covered:
- in create_json_source(), (if "unsupported" in msg) == False + check of the warning message
- in create_json_source(), if isinstance(e, UnboundLocalError) == False
- in render_bibliography(), (if render_outputs and len(render_outputs) > 0) == False
- in render_bibliography(), (if "'NoneType' object has no attribute 'render'" in str(e)) == False
- in render_bibliography(), the raise in branch except AttributeError as e: is not tested
- in render_bibliography(), the branch except Exception as e: is not tested, and the warning and return are not either
- in render_bibliography(), the return None, "Empty layout produced by bibliography generator." doesn't seem tested

---

---

# LLM

```
# tests/unit/adapters/test_citeproc_adapter.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Unit tests for the citeproc adapter layer."""

import logging
import warnings
from pathlib import Path
from typing import Any
from unittest.mock import MagicMock, patch

import pytest

from citecraft.adapters import CiteprocAdapter

# Dynamically load a minimalist valid XML CSL style mock derived from nature.csl
FIXTURE_DIR = Path(__file__).parent / "fixtures"
MOCK_CSL_STYLE = (FIXTURE_DIR / "mock_csl.xml").read_text(encoding="utf-8")


def test_create_json_source_success_with_unsupported_warning(
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify instantiation redirects citeproc internal warnings onto local logs."""
    csl_dict = {
        "id": "10.1000/xyz123",
        "type": "article-journal",
        "title": "Test Title",
        "published": "2026-01-01",  # Triggers citeproc unsupported variable warning
    }

    with caplog.at_level(logging.DEBUG):
        source, err = CiteprocAdapter.create_json_source(csl_dict, doi="10.1000/xyz123")

    assert source is not None
    assert err is None

    # Check warning redirection to debug log channel
    debug_records = [r for r in caplog.records if r.levelname == "DEBUG"]
    assert len(debug_records) == 1
    assert "citeproc_unsupported_fields_filtered" in getattr(
        debug_records[0],
        "event",
        "",
    )


def test_create_json_source_unexpected_warning(
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify handling of warnings that do not contain the term 'unsupported'."""

    def mock_init(*args: Any, **kwargs: Any) -> None:
        warnings.warn("Unexpected test warning detail", UserWarning)

    with (
        patch("citeproc.source.json.CiteProcJSON.__init__", side_effect=mock_init),
        caplog.at_level(logging.WARNING),
    ):
        _, _ = CiteprocAdapter.create_json_source(
            {"id": "10.1000/xyz", "type": "article-journal"},
            doi="10.1000/xyz",
        )
        assert "unexpected citeproc-py warning" in caplog.text.lower()


@pytest.mark.parametrize(
    "broken_csl_payload",
    [
        {"id": "10.1000/xyz123"},  # Missing entirely the required 'type'
        {"type": "article-journal"},  # Missing entirely the required 'id'
    ],
)
# WARNING: we don't the fields set to None, citeproc-py has another behavior that is
# inconsistent
def test_create_json_source_malformed_payloads(
    broken_csl_payload: dict[str, Any],
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify structural type and ID anomalies handle standard citeproc failures."""
    with caplog.at_level(logging.WARNING):
        source, err = CiteprocAdapter.create_json_source(
            broken_csl_payload,
            doi="10.1000/xyz123",
        )

    assert source is None
    assert err is not None
    assert len(caplog.records) == 1
    assert caplog.records[0].levelname == "WARNING"


def test_create_json_source_other_exception(
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify exception handling when non-UnboundLocalError triggers translation fail."""
    with (
        patch(
            "citeproc.source.json.CiteProcJSON.__init__",
            side_effect=KeyError("Missing key detail"),
        ),
        caplog.at_level(logging.WARNING),
    ):
        source, err = CiteprocAdapter.create_json_source(
            {"id": "10.1000/xyz", "type": "article-journal"},
            doi="10.1000/xyz",
        )
        assert source is None
        assert err is not None
        assert "Missing key detail" in err
        assert "UnboundLocalError" not in err


def test_parse_csl_style_success() -> None:
    """Verify successful parsing logic transformation onto native citeproc elements."""
    style, err = CiteprocAdapter.parse_csl_style(MOCK_CSL_STYLE, doi="10.1000/xyz123")
    assert style is not None
    assert err is None


def test_parse_csl_style_failure(caplog: pytest.LogCaptureFixture) -> None:
    """Verify broken layout XML code captures errors."""
    malformed_xml = "<style><invalid-unclosed-tag>"

    with caplog.at_level(logging.WARNING):
        style, err = CiteprocAdapter.parse_csl_style(
            malformed_xml,
            doi="10.1000/xyz123",
        )

    assert style is None
    assert err is not None
    assert len(caplog.records) == 1


def test_full_rendering_pipeline_success(caplog: pytest.LogCaptureFixture) -> None:
    """Verify end-to-end processing sequence through the adapter layers."""
    doi_key = "10.1000/xyz123"
    csl_dict = {
        "id": doi_key,
        "type": "article-journal",
        "title": "Glacier Melting Dynamics",
    }
    expected_test = (
        "1.Glacier Melting Dynamics.."  # WARNING Dependent on MOCK_CSL_STYLE
    )

    source, _ = CiteprocAdapter.create_json_source(csl_dict, doi=doi_key)
    style, _ = CiteprocAdapter.parse_csl_style(MOCK_CSL_STYLE, doi=doi_key)

    assert source is not None
    assert style is not None

    with caplog.at_level(logging.DEBUG):
        rendered_text, err = CiteprocAdapter.render_bibliography(
            style,
            source,
            item_id=doi_key,
            doi=doi_key,
        )

    assert err is None
    assert rendered_text == expected_test

    # Confirm final resolution tracing assertion
    ok_records = [
        r
        for r in caplog.records
        if r.levelname == "DEBUG"
        and "doi_local_resolution_success" in getattr(r, "event", "")
    ]
    assert len(ok_records) == 1


def test_render_bibliography_corrupted_github_style_handles_attribute_error(
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify that an AttributeError triggers a warning and fallback."""
    doi_key = "10.1000/xyz123"
    csl_dict = {
        "id": doi_key,
        "type": "article-journal",
        "title": "Glacier Melting Dynamics",
    }

    source, _ = CiteprocAdapter.create_json_source(csl_dict, doi=doi_key)
    style, _ = CiteprocAdapter.parse_csl_style(MOCK_CSL_STYLE, doi=doi_key)

    assert source is not None
    assert style is not None

    with (
        patch(
            "citeproc.CitationStylesBibliography.bibliography",
            side_effect=AttributeError("'NoneType' object has no attribute 'render'"),
        ),
        caplog.at_level(logging.WARNING),
    ):
        rendered_text, err = CiteprocAdapter.render_bibliography(
            style,
            source,
            item_id=doi_key,
            doi=doi_key,
        )

    assert rendered_text is None
    assert err is not None
    assert "hosted on GitHub appears to be corrupted" in err
    assert len(caplog.records) == 1
    record = caplog.records[0]
    assert record.levelname == "WARNING"
    assert getattr(record, "event", "") == "citeproc_github_style_file_corrupted"
    assert getattr(record, "doi", "") == doi_key


def test_render_bibliography_unrelated_attribute_error_bubbles_up() -> None:
    """Verify that unrelated AttributeErrors are not intercepted."""
    style = MagicMock()
    source = MagicMock()

    with (
        patch(
            "citeproc.CitationStylesBibliography.bibliography",
            side_effect=AttributeError("Unrelated AttributeError trigger"),
        ),
        pytest.raises(AttributeError, match="Unrelated AttributeError trigger"),
    ):
        CiteprocAdapter.render_bibliography(
            style,
            source,
            item_id="10.1000/unrelated",
            doi="10.1000/unrelated",
        )


def test_render_bibliography_empty_output_returns_error() -> None:
    """Verify fallback returning specific explanation when outputs list is empty."""
    style = MagicMock()
    source = MagicMock()

    with patch(
        "citeproc.CitationStylesBibliography.bibliography",
        return_value=[],
    ):
        rendered_text, err = CiteprocAdapter.render_bibliography(
            style,
            source,
            item_id="10.1000/empty",
            doi="10.1000/empty",
        )

    assert rendered_text is None
    assert err == "Empty layout produced by bibliography generator."


def test_render_bibliography_generic_exception_caught(
    caplog: pytest.LogCaptureFixture,
) -> None:
    """Verify that general rendering exceptions are gracefully caught and reported."""
    style = MagicMock()
    source = MagicMock()

    with (
        patch(
            "citeproc.CitationStylesBibliography.bibliography",
            side_effect=RuntimeError("Generic pipeline exception"),
        ),
        caplog.at_level(logging.WARNING),
    ):
        rendered_text, err = CiteprocAdapter.render_bibliography(
            style,
            source,
            item_id="10.1000/crash",
            doi="10.1000/crash",
        )

    assert rendered_text is None
    assert err == "Generic pipeline exception"
    assert "rendering pipeline crashed" in caplog.text.lower()
```

---

---

# ME

```
# src/citecraft/repositories/base_repository.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Base database repository implementing storage operations and schema enforcement."""

import logging
import sqlite3
from contextlib import suppress
from datetime import UTC, datetime
from pathlib import Path
from typing import TypeVar

from citecraft.network import (
    HTTPClientRegistry,
    get_http_client_registry,
)
from citecraft.schemas import BaseSchema
from citecraft.storage.db import load_records, save_records
from citecraft.utils import (
    AppConfig,
    get_config,
)

T = TypeVar("T")
logger = logging.getLogger(__name__)


class BaseRepository[T: BaseSchema]:
    """Base repository delivering local storage and schema enforcement."""

    def __init__(
        self,
        local_filename: str,
        model_class: type[T],
        config: AppConfig | None = None,
        api: str = "crossref",
        registry: HTTPClientRegistry | None = None,
    ) -> None:
        self.config: AppConfig = config or get_config()
        registry = registry or get_http_client_registry()
        self.http_client_wrapper = registry.get_client(api)
        self.headers: dict[str, str] = {
            "User-Agent": f"ManuscriptRefLister/1.0 (mailto:"
            f"{self.http_client_wrapper.email})",
        }
        self.local_filename: str = local_filename
        self.table_name: str = Path(local_filename).stem
        self._load_failed: bool = False
        self.model_class: type[T] = model_class
        self.records: list[T] = []

    def __len__(self) -> int:
        """Return the number of records currently loaded in memory."""
        return len(self.records)

    def deduplicate(self) -> None:
        """Remove duplicate records in-place based on their identity keys."""
        seen = set()
        unique_records = []

        for record in self.records:
            key = record.identity_key
            if key not in seen:
                seen.add(key)
                unique_records.append(record)

        self.records = unique_records

    def load_all(
        self,
        input_filepath: str | Path | None = None,
        *,
        raise_exception: bool = False,
    ) -> None:
        """Load and validate records into memory.

        The list of records of the object are set to [] if invalid.
        """
        path = Path(input_filepath or self.config.db_filepath).resolve()

        try:
            self.records = load_records(path, self.table_name, self.model_class)
            self._load_failed = False
        except (sqlite3.Error, TypeError, ValueError):
            logger.warning(
                "Failed validation or database corrupted for %s in SQLite file %s. "
                "Triggering database recovery and backup.",
                self.model_class.__name__,
                str(path),
                extra={
                    "status": "KO",
                    "event": "repository_validation_failed",
                    "model": self.model_class.__name__,
                    "filepath": str(path),
                },
            )

            if path.is_file():
                timestamp = datetime.now(UTC).strftime("%Y%m%d_%H%M%S")
                backup_filename = f"{path.stem}_corrupted_{timestamp}{path.suffix}"
                backup_path = path.with_name(backup_filename)
                try:
                    path.rename(backup_path)
                    logger.warning(
                        "Corrupted database backed up to: %s",
                        str(backup_path),
                        extra={"event": "database_corrupted_backup_created"},
                    )
                except OSError:
                    logger.exception(
                        "Failed to back up corrupted database",
                    )
                    with suppress(OSError):
                        path.unlink(missing_ok=True)

            self.records = []
            self._load_failed = True
            if raise_exception:
                raise

        logger.info(
            "Loaded %d records into memory from SQLite %s",
            len(self.records),
            str(path),
            extra={
                "status": "OK",
                "event": "repository_load_success",
                "record_count": len(self.records),
                "filepath": str(path),
            },
        )

    def save_all(self, output_filepath: str | Path | None = None) -> None:
        """Save records atomically to SQLite storage within a transaction."""
        target_path = Path(output_filepath or self.config.db_filepath).resolve()

        try:
            save_records(target_path, self.table_name, self.records, self.model_class)
            self._load_failed = False
        except (sqlite3.Error, OSError):
            logger.exception(
                "Failed to save records to SQLite at %s",
                str(target_path),
            )
            raise

        logger.info(
            "Saved %d records to SQLite %s",
            len(self.records),
            str(target_path),
            extra={
                "status": "OK",
                "event": "repository_save_success",
                "record_count": len(self.records),
                "filepath": str(target_path),
            },
        )

# tests/unit/repositories/test_base_repository.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Unit tests for the base database repository layer."""

import sqlite3
from collections.abc import Hashable
from pathlib import Path
from unittest.mock import patch

import pytest

from citecraft.repositories import BaseRepository
from citecraft.schemas import BaseSchema
from citecraft.storage.db import load_records, save_records
from citecraft.utils import AppConfig


class MockSchema(BaseSchema):
    """Simple schema for testing repository logic."""

    id: int
    content: str = "test"

    @property
    def identity_key(self) -> Hashable:
        """Return the unique identifier for deduplication testing."""
        return self.id


class MockRepository(BaseRepository[MockSchema]):
    """Concrete repository implementation mapping MockSchema elements."""


@pytest.fixture
def base_repo(test_config: AppConfig) -> MockRepository:
    """Provide a MockRepository instance utilizing the isolated test config."""
    return MockRepository("test_base_records.json", MockSchema, config=test_config)


def test_deduplicate_removes_repeats(base_repo: MockRepository) -> None:
    """Ensure records with identical identity_keys are removed, keeping the first."""
    base_repo.records = [
        MockSchema(id=1, content="first"),
        MockSchema(id=2, content="unique"),
        MockSchema(id=1, content="duplicate"),
    ]

    base_repo.deduplicate()

    assert len(base_repo) == 2
    assert base_repo.records[0].content == "first"
    assert [r.id for r in base_repo.records] == [1, 2]


@pytest.mark.parametrize(
    ("scenario", "expected_records_count", "expected_load_failed"),
    [
        ("valid", 2, False),
        ("corrupted", 0, True),
        ("missing", 0, False),
    ],
)
def test_load_all_scenarios(
    base_repo: MockRepository,
    scenario: str,
    expected_records_count: int,
    expected_load_failed: bool,
) -> None:
    """Verify load_all behaviors under standard database corruption states."""
    path = Path(base_repo.config.db_filepath)
    path.parent.mkdir(parents=True, exist_ok=True)

    if scenario == "valid":
        records = [MockSchema(id=1, content="A"), MockSchema(id=2, content="B")]
        save_records(path, base_repo.table_name, records, MockSchema)
    elif scenario == "corrupted":
        path.write_text("invalid corrupted sqlite binary header data", encoding="utf-8")
    elif scenario == "missing":
        if path.exists():
            path.unlink()

    base_repo.load_all()

    assert len(base_repo.records) == expected_records_count
    assert base_repo._load_failed is expected_load_failed

    if scenario == "corrupted":
        backup_files = list(path.parent.glob("*_corrupted_*.db"))
        assert len(backup_files) == 1
        assert not path.exists()


def test_save_all_atomic_success(base_repo: MockRepository, tmp_path: Path) -> None:
    """Verify save_all writes records cleanly to the SQLite database."""
    data = [MockSchema(id=1, content="saved")]
    base_repo.records = data
    target = tmp_path / "test_db.db"

    base_repo.save_all(output_filepath=target)

    assert target.exists()
    loaded = load_records(target, base_repo.table_name, MockSchema)
    assert len(loaded) == 1
    assert loaded[0].id == 1


def test_save_all_overwrite_existing(base_repo: MockRepository) -> None:
    """Verify that save_all correctly overwrites existing SQLite table records."""
    path = Path(base_repo.config.db_filepath)

    initial_records = [MockSchema(id=1, content="old")]
    save_records(path, base_repo.table_name, initial_records, MockSchema)

    new_record = MockSchema(id=99, content="new")
    base_repo.records = [new_record]
    base_repo.save_all()

    loaded = load_records(path, base_repo.table_name, MockSchema)
    assert len(loaded) == 1
    assert loaded[0].id == 99


def test_save_all_preserves_utf8(base_repo: MockRepository) -> None:
    """Ensure non-ASCII characters are saved as literal UTF-8 in the SQLite db."""
    special_content = "Lavé & Keddadouche — 2020"
    base_repo.records = [MockSchema(id=1, content=special_content)]

    base_repo.save_all()

    base_repo.load_all()
    assert base_repo.records[0].content == special_content


# =============================================================================
# COV TESTS: SPECIFIC IF/ELSE AND FALLBACK BRANCH COVERAGE
# =============================================================================


def test_load_all_backup_os_error_unlinks_file(
    base_repo: MockRepository,
) -> None:
    """Verify backup OSError unlinks corrupted file to ensure recovery."""
    path = Path(base_repo.config.db_filepath)
    path.parent.mkdir(parents=True, exist_ok=True)
    path.write_text("corrupted_database_mock_payload", encoding="utf-8")

    with patch.object(Path, "rename", side_effect=OSError("Rename blocked")):
        base_repo.load_all()

    assert base_repo._load_failed is True
    assert not path.exists()


def test_load_all_with_raise_exception(base_repo: MockRepository) -> None:
    """Verify load_all raises exception if raise_exception is explicitly True."""
    path = Path(base_repo.config.db_filepath)
    path.parent.mkdir(parents=True, exist_ok=True)
    path.write_text("corrupted_database_mock_payload", encoding="utf-8")

    with pytest.raises((sqlite3.Error, TypeError, ValueError)):
        base_repo.load_all(raise_exception=True)


def test_save_all_exception_raises_and_logs(
    base_repo: MockRepository,
) -> None:
    """Verify save_all failure triggers sqlite3.Error and exits cleanly."""
    base_repo.records = [MockSchema(id=1)]
    with (
        patch(
            "citecraft.repositories.base_repository.save_records",
            side_effect=sqlite3.OperationalError("Mock saving database error"),
        ),
        pytest.raises(sqlite3.Error),
    ):
        base_repo.save_all()
```

This branch is not covered:
- in load_all(), if path.is_file() == False

---

---

# LLM

```
# tests/unit/repositories/test_base_repository.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Unit tests for the base database repository layer."""

import sqlite3
from collections.abc import Hashable
from pathlib import Path
from unittest.mock import patch

import pytest

from citecraft.repositories import BaseRepository
from citecraft.schemas import BaseSchema
from citecraft.storage.db import load_records, save_records
from citecraft.utils import AppConfig


class MockSchema(BaseSchema):
    """Simple schema for testing repository logic."""

    id: int
    content: str = "test"

    @property
    def identity_key(self) -> Hashable:
        """Return the unique identifier for deduplication testing."""
        return self.id


class MockRepository(BaseRepository[MockSchema]):
    """Concrete repository implementation mapping MockSchema elements."""


@pytest.fixture
def base_repo(test_config: AppConfig) -> MockRepository:
    """Provide a MockRepository instance utilizing the isolated test config."""
    return MockRepository("test_base_records.json", MockSchema, config=test_config)


def test_deduplicate_removes_repeats(base_repo: MockRepository) -> None:
    """Ensure records with identical identity_keys are removed, keeping the first."""
    base_repo.records = [
        MockSchema(id=1, content="first"),
        MockSchema(id=2, content="unique"),
        MockSchema(id=1, content="duplicate"),
    ]

    base_repo.deduplicate()

    assert len(base_repo) == 2
    assert base_repo.records[0].content == "first"
    assert [r.id for r in base_repo.records] == [1, 2]


@pytest.mark.parametrize(
    ("scenario", "expected_records_count", "expected_load_failed"),
    [
        ("valid", 2, False),
        ("corrupted", 0, True),
        ("missing", 0, False),
    ],
)
def test_load_all_scenarios(
    base_repo: MockRepository,
    scenario: str,
    expected_records_count: int,
    expected_load_failed: bool,
) -> None:
    """Verify load_all behaviors under standard database corruption states."""
    path = Path(base_repo.config.db_filepath)
    path.parent.mkdir(parents=True, exist_ok=True)

    if scenario == "valid":
        records = [MockSchema(id=1, content="A"), MockSchema(id=2, content="B")]
        save_records(path, base_repo.table_name, records, MockSchema)
    elif scenario == "corrupted":
        path.write_text("invalid corrupted sqlite binary header data", encoding="utf-8")
    elif scenario == "missing":
        if path.exists():
            path.unlink()

    base_repo.load_all()

    assert len(base_repo.records) == expected_records_count
    assert base_repo._load_failed is expected_load_failed

    if scenario == "corrupted":
        backup_files = list(path.parent.glob("*_corrupted_*.db"))
        assert len(backup_files) == 1
        assert not path.exists()


def test_save_all_atomic_success(base_repo: MockRepository, tmp_path: Path) -> None:
    """Verify save_all writes records cleanly to the SQLite database."""
    data = [MockSchema(id=1, content="saved")]
    base_repo.records = data
    target = tmp_path / "test_db.db"

    base_repo.save_all(output_filepath=target)

    assert target.exists()
    loaded = load_records(target, base_repo.table_name, MockSchema)
    assert len(loaded) == 1
    assert loaded[0].id == 1


def test_save_all_overwrite_existing(base_repo: MockRepository) -> None:
    """Verify that save_all correctly overwrites existing SQLite table records."""
    path = Path(base_repo.config.db_filepath)

    initial_records = [MockSchema(id=1, content="old")]
    save_records(path, base_repo.table_name, initial_records, MockSchema)

    new_record = MockSchema(id=99, content="new")
    base_repo.records = [new_record]
    base_repo.save_all()

    loaded = load_records(path, base_repo.table_name, MockSchema)
    assert len(loaded) == 1
    assert loaded[0].id == 99


def test_save_all_preserves_utf8(base_repo: MockRepository) -> None:
    """Ensure non-ASCII characters are saved as literal UTF-8 in the SQLite db."""
    special_content = "Lavé & Keddadouche — 2020"
    base_repo.records = [MockSchema(id=1, content=special_content)]

    base_repo.save_all()

    base_repo.load_all()
    assert base_repo.records[0].content == special_content


# =============================================================================
# COV TESTS: SPECIFIC IF/ELSE AND FALLBACK BRANCH COVERAGE
# =============================================================================


def test_load_all_backup_os_error_unlinks_file(
    base_repo: MockRepository,
) -> None:
    """Verify backup OSError unlinks corrupted file to ensure recovery."""
    path = Path(base_repo.config.db_filepath)
    path.parent.mkdir(parents=True, exist_ok=True)
    path.write_text("corrupted_database_mock_payload", encoding="utf-8")

    with patch.object(Path, "rename", side_effect=OSError("Rename blocked")):
        base_repo.load_all()

    assert base_repo._load_failed is True
    assert not path.exists()


def test_load_all_with_raise_exception(base_repo: MockRepository) -> None:
    """Verify load_all raises exception if raise_exception is explicitly True."""
    path = Path(base_repo.config.db_filepath)
    path.parent.mkdir(parents=True, exist_ok=True)
    path.write_text("corrupted_database_mock_payload", encoding="utf-8")

    with pytest.raises((sqlite3.Error, TypeError, ValueError)):
        base_repo.load_all(raise_exception=True)


def test_load_all_when_path_is_not_file(
    base_repo: MockRepository,
    tmp_path: Path,
) -> None:
    """Verify load_all skips backup logic if database path is not a file."""
    # A directory path is not a file (path.is_file() == False)
    dir_path = tmp_path / "not_a_file_directory"
    dir_path.mkdir()

    base_repo.load_all(input_filepath=dir_path)

    assert base_repo._load_failed is True
    assert len(base_repo.records) == 0

    # No backup files should exist because is_file() evaluated to False
    backup_files = list(dir_path.parent.glob("*_corrupted_*.db"))
    assert len(backup_files) == 0


def test_save_all_exception_raises_and_logs(
    base_repo: MockRepository,
) -> None:
    """Verify save_all failure triggers sqlite3.Error and exits cleanly."""
    base_repo.records = [MockSchema(id=1)]
    with (
        patch(
            "citecraft.repositories.base_repository.save_records",
            side_effect=sqlite3.OperationalError("Mock saving database error"),
        ),
        pytest.raises(sqlite3.Error),
    ):
        base_repo.save_all()
```

---

---

# ME

```
# src/citecraft/ui/progress_bar_context.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Asynchronous CLI progress bar visualization and logging interception hooks."""

import logging
import sys
import threading
import time
from collections.abc import Callable
from types import TracebackType
from typing import Any, Self

from citecraft.core import ProgressStep


class LogInterceptor(logging.Handler):
    """Handler that intercepts and redirects logs above the progress bar."""

    def __init__(self, draw_callback: Callable[[], None]) -> None:
        super().__init__()
        self.draw_callback: Callable[[], None] = draw_callback

    def emit(self, record: logging.LogRecord) -> None:
        """Format and write incoming log records safely above the active bar line."""
        try:
            msg = self.format(record)
            # \r = retour chariot, \033[K = efface le reste de la ligne courante
            sys.stderr.write(f"\r\033[K{msg}\n")
            sys.stderr.flush()
            self.draw_callback()
        except Exception:  # noqa: BLE001 Logging should not crash the application
            self.handleError(record)


class ProgressBarContext:
    """Context manager responsible for the lifecycle of the CLI progress bar.

    It manages an asynchronous background thread that refreshes the console
    at a 1 Hz frequency, preventing the terminal from appearing frozen during
    long-running IO tasks (e.g., Crossref API lookups).

    If verbose_level > 0, the context remains passive and does not alter
    the standard output stream.
    """

    def __init__(self, verbose_level: int = 0, bar_width: int = 30) -> None:
        self.verbose_level: int = verbose_level
        self.bar_width: int = bar_width
        self.is_active: bool = verbose_level == 0

        # Thread-safe communication state
        self._lock: threading.Lock = threading.Lock()
        self._state: dict[str, Any] = {
            "message": "Initializing...",
            "current_step": 0,
            "total_steps": 5,
            "running": False,
        }

        self._start_time: float = 0.0
        self._ticker_thread: threading.Thread | None = None
        self._stop_event: threading.Event = threading.Event()

        self._old_handlers: list[logging.Handler] = []
        self._custom_handler: LogInterceptor | None = None

    def __enter__(self) -> Self:
        """Start the background progress-bar refresh loop."""
        if self.is_active:
            self._start_time = time.time()
            self._state["running"] = True
            self._stop_event.clear()

            # Divert logging to interceptor
            root_logger = logging.getLogger()
            self._old_handlers = root_logger.handlers[:]
            for h in self._old_handlers:
                root_logger.removeHandler(h)

            self._custom_handler = LogInterceptor(draw_callback=self._draw_line)
            self._custom_handler.setFormatter(
                logging.Formatter(
                    "%(asctime)s - %(name)s - %(levelname)s - %(message)s",
                ),
            )
            root_logger.addHandler(self._custom_handler)

            # Start background worker ticker thread
            self._ticker_thread = threading.Thread(
                target=self._loop_render,
                daemon=True,
            )
            self._ticker_thread.start()
        return self

    def __exit__(
        self,
        exc_type: type[BaseException] | None,
        exc_val: BaseException | None,
        exc_tb: TracebackType | None,
    ) -> None:
        """Ensure clean thread teardown and restoration of logging configurations."""
        if not self.is_active:
            return

        # Signal thread shutdown immediately
        with self._lock:
            self._state["running"] = False

        # Instantly wake up the thread loop to prevent test execution deadlocks
        self._stop_event.set()

        if self._ticker_thread:
            self._ticker_thread.join(timeout=1.0)

        # Restore handlers
        root_logger = logging.getLogger()
        if self._custom_handler:
            root_logger.removeHandler(self._custom_handler)
        for h in self._old_handlers:
            root_logger.addHandler(h)

        # Evaluate final terminal status
        if exc_type is None:
            with self._lock:
                self._state["current_step"] = self._state["total_steps"]
                self._state["message"] = "Completed."
            self._draw_line()
            sys.stderr.write("\n")
        else:
            # Application crash occurred: break line instantly to keep traceback clean
            sys.stderr.write("\n")

        sys.stderr.flush()

    def update(self, step: ProgressStep) -> None:
        """Update current state thread-safely."""
        if not self.is_active:
            return

        should_render = False
        with self._lock:
            self._state["message"] = step.message
            self._state["total_steps"] = step.total
            if step.status == "completed":
                self._state["current_step"] = step.current
                should_render = True
        # Force synchronous line redraw on step boundary resolution
        # Fix: Call draw outside of the lock context to prevent re-entrancy deadlocks
        if should_render:
            self._draw_line()

    def generate_bar_string(self, current: int, total: int, elapsed_time: float) -> str:
        """Compute the progress bar visual format and remaining time estimation."""
        percent = int((current / total) * 100) if total > 0 else 0
        filled_length = int(self.bar_width * current // total) if total > 0 else 0

        # Colors cyan block
        fill_char = "\033[36m█\033[0m"
        empty_char = "░"
        bar = (fill_char * filled_length) + (
            empty_char * (self.bar_width - filled_length)
        )

        if current == 0 or total == 0:
            eta_str = "--:--"
        elif current == total:
            eta_str = "00:00"
        else:
            remaining_steps = total - current
            estimated_remaining_seconds = int(
                (remaining_steps * elapsed_time) / current,
            )
            eta_min, eta_sec = divmod(estimated_remaining_seconds, 60)
            eta_str = f"{eta_min:02d}:{eta_sec:02d}"

        return f"[{bar}] {percent}% (ETA: {eta_str})"

    def _loop_render(self) -> None:
        """Refresh progress visualization at a 1 Hz interval."""
        while not self._stop_event.is_set():
            with self._lock:
                if not self._state["running"]:
                    break
            self._draw_line()
            # Sleeps for 1s or wakes up immediately if stop_event.set() is called
            if self._stop_event.wait(timeout=1.0):
                break

    def _draw_line(self) -> None:
        """Draw a single carriage-return progress line to stderr."""
        with self._lock:
            current = self._state["current_step"]
            total = self._state["total_steps"]
            msg = self._state["message"]

        elapsed = time.time() - self._start_time
        bar_component = self.generate_bar_string(current, total, elapsed)

        # Format layout matching original line configuration
        elapsed_int = int(elapsed)
        minutes, seconds = divmod(elapsed_int, 60)
        time_str = f"[{minutes:02d}:{seconds:02d}]"

        # Stay on a unique stable line
        line = f"\r\033[K{time_str} {msg:<55} {bar_component}"
        sys.stderr.write(line)
        sys.stderr.flush()

# tests/unit/ui/test_progress_bar_context.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Unit tests validating CLI progress bar layout formatting, lifecycle, and logs."""

import io
import logging
import sys
import time

import pytest

from citecraft.core import ProgressStep
from citecraft.ui.progress_bar_context import ProgressBarContext


def test_generate_bar_string_initial_state() -> None:
    """Verify bar string formatting when no steps have been processed."""
    ctx = ProgressBarContext(verbose_level=0, bar_width=10)
    # current = 0, total = 4, elapsed = 5.0
    bar_str = ctx.generate_bar_string(0, 4, 5.0)

    assert "░" * 10 in bar_str
    assert "0%" in bar_str
    assert "ETA: --:--" in bar_str


def test_generate_bar_string_zero_elapsed_time() -> None:
    """Verify that elapsed time at zero does not trigger ZeroDivisionError."""
    ctx = ProgressBarContext(verbose_level=0, bar_width=10)
    # Boundary: 0 elapsed time, 0 step achieved
    bar_str_initial = ctx.generate_bar_string(0, 4, 0.0)
    assert "ETA: --:--" in bar_str_initial

    # Boundary: step achieved but 0 elapsed time
    bar_str_progress = ctx.generate_bar_string(2, 4, 0.0)
    assert "ETA: 00:00" in bar_str_progress


def test_generate_bar_string_mid_progress() -> None:
    """Verify ETA calculation and fill ratio at progress midpoint."""
    ctx = ProgressBarContext(verbose_level=0, bar_width=10)
    # 2 out of 4 steps done in 10 seconds -> 5 seconds per step average.
    # 2 steps remaining * 5s = 10s remaining -> ETA: 00:10
    bar_str = ctx.generate_bar_string(2, 4, 10.0)

    # 50% fill: 5 filled chars, 5 empty chars
    expected_fill = "\033[36m█\033[0m" * 5 + "░" * 5
    assert expected_fill in bar_str
    assert "50%" in bar_str
    assert "ETA: 00:10" in bar_str


def test_generate_bar_string_completed_state() -> None:
    """Verify that completing all steps forces ETA to zero."""
    ctx = ProgressBarContext(verbose_level=0, bar_width=10)
    bar_str = ctx.generate_bar_string(4, 4, 15.0)

    assert "\033[36m█\033[0m" * 10 in bar_str
    assert "100%" in bar_str
    assert "ETA: 00:00" in bar_str


def test_progress_bar_passive_mode_when_verbose() -> None:
    """Ensure context remains completely silent if verbose level is activated."""
    with ProgressBarContext(verbose_level=1) as ctx:
        assert ctx.is_active is False
        assert ctx._ticker_thread is None

        # Callbacks should execute as no-ops safely
        step = ProgressStep(
            step_name="parsing",
            current=1,
            total=4,
            message="Test",
            status="started",
        )
        ctx.update(step)
        assert ctx._state["current_step"] == 0  # State unchanged


def test_progress_lifecycle_and_render(monkeypatch: pytest.MonkeyPatch) -> None:
    """Test standard execution lifecycle rendering on sys.stderr."""
    fake_stderr = io.StringIO()
    # Replace sys.stderr safely during this test execution
    monkeypatch.setattr(sys, "stderr", fake_stderr)
    # We use a passive manual execution by mocking or controlling time to avoid 1Hz
    # sleep race conditions
    ctx = ProgressBarContext(verbose_level=0, bar_width=10)

    with ctx:
        # Simulate core pipeline calling the update handler
        ctx.update(
            ProgressStep(
                step_name="parsing",
                current=1,
                total=4,
                message="Parsing documents",
                status="completed",
            ),
        )
        # Leave brief window for background thread to execute one loop cleanly
        time.sleep(0.05)

    output = fake_stderr.getvalue()

    # Assert specific visual structures match our designed layout
    assert "Initializing..." in output
    assert "Parsing documents" in output
    assert "25%" in output
    assert "Completed." in output


def test_progress_lifecycle_on_exception(monkeypatch: pytest.MonkeyPatch) -> None:
    """Verify that a pipeline crash forces a newline without printing 'Completed.'."""
    fake_stderr = io.StringIO()
    monkeypatch.setattr(sys, "stderr", fake_stderr)
    ctx = ProgressBarContext(verbose_level=0, bar_width=10)

    # Converting to PT012 puts the test in fail.
    with pytest.raises(ValueError, match="Pipeline Failure"), ctx:  # noqa: PT012
        ctx.update(
            ProgressStep(
                step_name="parsing",
                current=0,
                total=4,
                message="Crashing step",
                status="started",
            ),
        )
        err_msg = "Pipeline Failure"
        raise ValueError(err_msg)

    output = fake_stderr.getvalue()
    assert "Completed." not in output
    # Ends with a raw newline to clear the line for the incoming traceback
    assert output.endswith("\n")


def test_log_clears_progress_bar_line(capsys: pytest.CaptureFixture[str]) -> None:
    """Verify that logging correctly erase progress bar lines before emitting logs."""
    # 1. Setup local logging infrastructure connected to root for context interception
    test_logger = logging.getLogger("citecraft.integ_ui")
    test_logger.setLevel(logging.WARNING)
    test_logger.propagate = (
        True  # Must propagate to root so ProgressBarContext can intercept it
    )

    try:
        # 2. Emulate an active progress UI layout state
        ctx = ProgressBarContext(verbose_level=0, bar_width=10)

        with ctx:
            # Force an explicit line draw to establish the terminal state
            # "Initializing..."
            ctx._draw_line()

            # Emit a warning log midway through execution
            test_logger.warning("Network timeout encountered, retrying...")

            # Draw line again to mimic the next 1Hz tick iteration
            ctx._draw_line()

        captured = capsys.readouterr()
        err_output = captured.err

        # 3. Assert structural integrity of the stream sequence
        # The output must contain the clear line carriage return token before the log
        # Note: In Option B, LogInterceptor applies formatting or the context redraws.
        assert "Network timeout encountered, retrying..." in err_output

        # Verify ANSI erase sequence
        # Log mustn't overwrite remaining progressbar
        assert "\r\033[K" in err_output or "\r\x1b[K" in err_output

        # Verify the progress bar successfully reprinted its states during the cycle
        assert "Initializing..." in err_output
        assert "Completed." in err_output

    finally:
        # Ensure clean state for subsequent tests
        test_logger.propagate = False


def test_logging_handlers_are_cleaned_on_exception() -> None:
    """Verify that ProgressBarContext removes handlers during crashes."""
    root_logger = logging.getLogger()
    initial_handlers = list(root_logger.handlers)

    ctx = ProgressBarContext(verbose_level=0, bar_width=10)

    with ctx:
        assert any(
            x.__class__.__name__ == "LogInterceptor" for x in root_logger.handlers
        )
        err_msg = "Forced pipeline crash"
        with pytest.raises(RuntimeError, match="Forced pipeline crash"):
            raise RuntimeError(err_msg)

    # After-crash verification: initial state must be restored.
    assert root_logger.handlers == initial_handlers
```

these branches are not covered:
- in emit(), except Exception:
- in __exit__(),  if self._ticker_thread == False
- in __exit__(),  if self._custom_handler == False
- in _loop_render(), while not self._stop_event.is_set() == False
- in _loop_render(), if not self._state["running"] == True
- in _loop_render(), if self._stop_event.wait(timeout=1.0) == False

---

---

# LLM

```
# tests/unit/ui/test_progress_bar_context.py
# SPDX-FileCopyrightText: 2026 Sebastien Lenard <sebastien.lenard@gmail.com> and Contributors
# SPDX-License-Identifier: Apache-2.0
"""Unit tests validating CLI progress bar layout formatting, lifecycle, and logs."""

import io
import logging
import sys
import time
from typing import Any
from unittest.mock import MagicMock, patch

import pytest

from citecraft.core import ProgressStep
from citecraft.ui.progress_bar_context import LogInterceptor, ProgressBarContext


def test_generate_bar_string_initial_state() -> None:
    """Verify bar string formatting when no steps have been processed."""
    ctx = ProgressBarContext(verbose_level=0, bar_width=10)
    # current = 0, total = 4, elapsed = 5.0
    bar_str = ctx.generate_bar_string(0, 4, 5.0)

    assert "░" * 10 in bar_str
    assert "0%" in bar_str
    assert "ETA: --:--" in bar_str


def test_generate_bar_string_zero_elapsed_time() -> None:
    """Verify that elapsed time at zero does not trigger ZeroDivisionError."""
    ctx = ProgressBarContext(verbose_level=0, bar_width=10)
    # Boundary: 0 elapsed time, 0 step achieved
    bar_str_initial = ctx.generate_bar_string(0, 4, 0.0)
    assert "ETA: --:--" in bar_str_initial

    # Boundary: step achieved but 0 elapsed time
    bar_str_progress = ctx.generate_bar_string(2, 4, 0.0)
    assert "ETA: 00:00" in bar_str_progress


def test_generate_bar_string_mid_progress() -> None:
    """Verify ETA calculation and fill ratio at progress midpoint."""
    ctx = ProgressBarContext(verbose_level=0, bar_width=10)
    # 2 out of 4 steps done in 10 seconds -> 5 seconds per step average.
    # 2 steps remaining * 5s = 10s remaining -> ETA: 00:10
    bar_str = ctx.generate_bar_string(2, 4, 10.0)

    # 50% fill: 5 filled chars, 5 empty chars
    expected_fill = "\033[36m█\033[0m" * 5 + "░" * 5
    assert expected_fill in bar_str
    assert "50%" in bar_str
    assert "ETA: 00:10" in bar_str


def test_generate_bar_string_completed_state() -> None:
    """Verify that completing all steps forces ETA to zero."""
    ctx = ProgressBarContext(verbose_level=0, bar_width=10)
    bar_str = ctx.generate_bar_string(4, 4, 15.0)

    assert "\033[36m█\033[0m" * 10 in bar_str
    assert "100%" in bar_str
    assert "ETA: 00:00" in bar_str


def test_progress_bar_passive_mode_when_verbose() -> None:
    """Ensure context remains completely silent if verbose level is activated."""
    with ProgressBarContext(verbose_level=1) as ctx:
        assert ctx.is_active is False
        assert ctx._ticker_thread is None

        # Callbacks should execute as no-ops safely
        step = ProgressStep(
            step_name="parsing",
            current=1,
            total=4,
            message="Test",
            status="started",
        )
        ctx.update(step)
        assert ctx._state["current_step"] == 0  # State unchanged


def test_progress_lifecycle_and_render(monkeypatch: pytest.MonkeyPatch) -> None:
    """Test standard execution lifecycle rendering on sys.stderr."""
    fake_stderr = io.StringIO()
    # Replace sys.stderr safely during this test execution
    monkeypatch.setattr(sys, "stderr", fake_stderr)
    # We use a passive manual execution by mocking or controlling time to avoid 1Hz
    # sleep race conditions
    ctx = ProgressBarContext(verbose_level=0, bar_width=10)

    with ctx:
        # Simulate core pipeline calling the update handler
        ctx.update(
            ProgressStep(
                step_name="parsing",
                current=1,
                total=4,
                message="Parsing documents",
                status="completed",
            ),
        )
        # Leave brief window for background thread to execute one loop cleanly
        time.sleep(0.05)

    output = fake_stderr.getvalue()

    # Assert specific visual structures match our designed layout
    assert "Initializing..." in output
    assert "Parsing documents" in output
    assert "25%" in output
    assert "Completed." in output


def test_progress_lifecycle_on_exception(monkeypatch: pytest.MonkeyPatch) -> None:
    """Verify that a pipeline crash forces a newline without printing 'Completed.'."""
    fake_stderr = io.StringIO()
    monkeypatch.setattr(sys, "stderr", fake_stderr)
    ctx = ProgressBarContext(verbose_level=0, bar_width=10)

    # Converting to PT012 puts the test in fail.
    with pytest.raises(ValueError, match="Pipeline Failure"), ctx:  # noqa: PT012
        ctx.update(
            ProgressStep(
                step_name="parsing",
                current=0,
                total=4,
                message="Crashing step",
                status="started",
            ),
        )
        err_msg = "Pipeline Failure"
        raise ValueError(err_msg)

    output = fake_stderr.getvalue()
    assert "Completed." not in output
    # Ends with a raw newline to clear the line for the incoming traceback
    assert output.endswith("\n")


def test_log_clears_progress_bar_line(capsys: pytest.CaptureFixture[str]) -> None:
    """Verify that logging correctly erase progress bar lines before emitting logs."""
    # 1. Setup local logging infrastructure connected to root for context interception
    test_logger = logging.getLogger("citecraft.integ_ui")
    test_logger.setLevel(logging.WARNING)
    test_logger.propagate = (
        True  # Must propagate to root so ProgressBarContext can intercept it
    )

    try:
        # 2. Emulate an active progress UI layout state
        ctx = ProgressBarContext(verbose_level=0, bar_width=10)

        with ctx:
            # Force an explicit line draw to establish the terminal state
            # "Initializing..."
            ctx._draw_line()

            # Emit a warning log midway through execution
            test_logger.warning("Network timeout encountered, retrying...")

            # Draw line again to mimic the next 1Hz tick iteration
            ctx._draw_line()

        captured = capsys.readouterr()
        err_output = captured.err

        # 3. Assert structural integrity of the stream sequence
        # The output must contain the clear line carriage return token before the log
        # Note: In Option B, LogInterceptor applies formatting or the context redraws.
        assert "Network timeout encountered, retrying..." in err_output

        # Verify ANSI erase sequence
        # Log mustn't overwrite remaining progressbar
        assert "\r\033[K" in err_output or "\r\x1b[K" in err_output

        # Verify the progress bar successfully reprinted its states during the cycle
        assert "Initializing..." in err_output
        assert "Completed." in err_output

    finally:
        # Ensure clean state for subsequent tests
        test_logger.propagate = False


def test_logging_handlers_are_cleaned_on_exception() -> None:
    """Verify that ProgressBarContext removes handlers during crashes."""
    root_logger = logging.getLogger()
    initial_handlers = list(root_logger.handlers)

    ctx = ProgressBarContext(verbose_level=0, bar_width=10)

    with ctx:
        assert any(
            x.__class__.__name__ == "LogInterceptor" for x in root_logger.handlers
        )
        err_msg = "Forced pipeline crash"
        with pytest.raises(RuntimeError, match="Forced pipeline crash"):
            raise RuntimeError(err_msg)

    # After-crash verification: initial state must be restored.
    assert root_logger.handlers == initial_handlers


# =============================================================================
# COV TESTS: SPECIFIC HANDLER AND TIMER THREAD ROADMAPS
# =============================================================================


def test_log_interceptor_emit_exception_handled() -> None:
    """Verify LogInterceptor safely recovers when standard streams are locked."""
    interceptor = LogInterceptor(draw_callback=lambda: None)
    record = logging.LogRecord(
        name="test",
        level=logging.INFO,
        pathname="path",
        lineno=10,
        msg="log message content",
        args=(),
        exc_info=None,
    )

    with (
        patch("sys.stderr.write", side_effect=RuntimeError("Stream blocked")),
        patch.object(interceptor, "handleError") as mock_handle_error,
    ):
        interceptor.emit(record)
        mock_handle_error.assert_called_once_with(record)


def test_exit_with_falsy_handlers_and_threads() -> None:
    """Verify __exit__ successfully clears context even if handles or threads are None."""
    ctx = ProgressBarContext(verbose_level=0)
    ctx.is_active = True
    ctx._ticker_thread = None
    ctx._custom_handler = None

    # This call should finalize cleanly and execute zero-ops on falsy values
    ctx.__exit__(None, None, None)


def test_loop_render_stops_immediately_if_stop_event_set() -> None:
    """Verify loop exits instantly if the shutdown signal was pre-configured."""
    ctx = ProgressBarContext(verbose_level=0)
    ctx._stop_event.set()

    with patch.object(ctx, "_draw_line") as mock_draw:
        ctx._loop_render()
        mock_draw.assert_not_called()


def test_loop_render_breaks_if_not_running() -> None:
    """Verify background task breaks immediately when context running state is deactivated."""
    ctx = ProgressBarContext(verbose_level=0)
    ctx._state["running"] = False
    ctx._stop_event.clear()

    with patch.object(ctx, "_draw_line") as mock_draw:
        ctx._loop_render()
        mock_draw.assert_not_called()


def test_loop_render_continues_on_wait_timeout() -> None:
    """Verify background task processes next loop iteration if wait timeout expires."""
    ctx = ProgressBarContext(verbose_level=0)
    ctx._state["running"] = True
    ctx._stop_event.clear()

    call_count = 0

    def mock_draw() -> None:
        nonlocal call_count
        call_count += 1
        if call_count >= 2:
            # Safely trigger loop break on next iteration
            ctx._state["running"] = False

    with (
        patch.object(ctx, "_draw_line", side_effect=mock_draw),
        patch.object(ctx._stop_event, "wait", return_value=False) as mock_wait,
    ):
        ctx._loop_render()
        assert call_count == 2
        mock_wait.assert_called()
```

---

---

# LLM