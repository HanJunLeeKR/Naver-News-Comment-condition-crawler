# Naver News & Comment Collector

Collect Naver news articles and **all of their comments (including replies)** for any period, keyword and outlet — and record each comment's **moderation status**: author-deleted, clean-bot, deleted for violating the operating policy, taken down for rights infringement, or still exposed. For handled comments, the tool also records how long the comment stayed visible before it was handled (Chae & Lee, 2026).

> **Update (2026-10-03): the notebook was rewritten.** Naver changed its news search from numbered pages to infinite scroll in September 2026, which broke older versions. The new version no longer uses Selenium or ChromeDriver. See the [changelog](#changelog).

## Quick start

1. Download `naver_news_comment_collector.ipynb` and open it in Jupyter (or VS Code).
2. Run the first cell once to install the libraries, then restart the kernel:
   `%pip install requests beautifulsoup4 pandas tqdm xlsxwriter ipywidgets`
3. Edit the settings cell, then press **Run All**.

```python
검색어   = ["5·18"]                  # several keywords allowed: ["5.18", "5·18"]; [] = every article of the outlet
언론사   = ["연합뉴스", "조선일보"]    # outlet names, or "전체" (79 outlets)
시작일   = "2025-05-17"
종료일   = "2025-05-19"
저장폴더 = r"C:\naver_news_output"    # where results are saved
수집방식 = "전량"                     # "전량" (complete) or "검색" (search)
댓글수집 = True                       # True: articles + comments + replies / False: articles only
```

If the run stops (error, power loss, Naver temporarily blocks you), **run the notebook again with the same settings**. Finished parts are skipped and collection resumes where it stopped.

## Two collection modes

| | `수집방식 = "전량"` (complete) | `수집방식 = "검색"` (search) |
|---|---|---|
| How | Opens every article of the outlet in the period, by article number, and looks for the keyword in the title, body and photo captions | Uses Naver's news search, one day at a time |
| Missing articles | None (articles are enumerated, not searched) | Articles whose keyword appears **only in a photo caption** are not returned by Naver search |
| Search-server blocks (HTTP 403/429) | Rarely (does not use the search server) | More likely on long periods and many outlets |
| Speed | Depends on the number of articles of the outlet, not on the keyword. Each downloaded article is cached in `_cache`, so **running another keyword later does not download again** | Fast for short periods |
| Recommended for | Long periods, many outlets, "no article may be missed" | Quick checks over short periods |

## What is collected

- **Articles** (`articles.csv` + an Excel-safe `.xlsx` copy): one row per article — title, subtitle, body, photo captions, outlet, times, reporter name / e-mail / journalist ID, the publisher's summary box (`summary_box`), comment counts, the ten article-reaction counts (`react_*`), and — when Naver shows them — the commenters' gender and age shares (`commenter_male_pct`, `commenter_age10_pct` … `commenter_age70_pct`).
- **Comments and replies** (`comments/…csv` + `.xlsx`): one row per comment or reply — text, like / dislike counts, author mark number, write time, and the moderation status (`action_type`) with the time until it was handled.
- **Completeness check** (`check_…csv`) and **run report** (`수집보고서.txt / .csv`): for every article, the number of comments collected is compared with the number Naver itself reports.

Moderation status (`action_type`) follows Naver's own comment-widget logic: author-deleted (`작성자삭제`), deleted for violating the operating policy (`운영규정미준수삭제`), taken down for rights infringement (`권리침해게시중단`), deleted by the content provider (`콘텐츠제공자삭제`), clean-bot hidden (`클린봇`), otherwise exposed. Column definitions: see `naver_news_crawler_field_reference` (being updated for the new columns).

## Known limitations

- Search mode cannot find articles whose keyword appears only in a photo caption (use complete mode).
- A single Naver search returns at most about 600 results. The collector searches one day at a time; if a day reaches the limit it prints a warning. When Naver matches loosely (e.g. "5·18" also matches articles with "5" and "18"), the results beyond the limit are usually non-matching articles, and the default strict filter keeps only articles that contain the keyword literally.
- Sports and entertainment articles have **no comment section** on Naver, so only article fields are collected for them.
- Commenter gender / age statistics are shown by Naver only for articles with enough comments; otherwise those columns are left blank (blank ≠ 0).
- The article-reaction buttons changed over time (until about 2022: like / warm / sad / angry / want-follow-up; later: useful / wow / touched / analytic / recommend). All ten are stored in separate columns, so check which ones apply to the period you study.
- Naver changes its pages and APIs without notice. If something stops working, please open an issue.

## If you get HTTP 403 (temporary block)

The collector pauses all requests (1 → 3 → 5 → 10 → 15 → 30 minutes), slows down, and never silently skips a period; an outlet that could not be finished is not saved, so a rerun completes it. For long periods or many outlets, use complete mode. A debugging note on Naver's `more` search API (including the `cluster_rank` parameter) is kept in the project docs.

## Responsible use

Comments contain personal data. Please check Naver's terms of service and its current crawling policy (Naver has publicly restricted automated collection, notably by AI crawlers, since 2025) and your institution's research-ethics requirements before collecting, and **do not publish raw comment data or author identifiers**. The author mark number is provided only to distinguish authors within a dataset. This tool is provided for research purposes, without warranty, under the MIT license.

## Environment

Developed with Python 3.11.8. Tested on Windows (3.11.11), Mac (3.13.15) and Linux (3.11.15). Libraries: `requests`, `beautifulsoup4`, `pandas`, `tqdm`, `xlsxwriter`, `ipywidgets` (installed by the first cell).

## Changelog

- **2026/10/03** — Rewritten after Naver moved news search to infinite scroll (the old search-page method stopped working). New: complete mode, replies, moderation classification following Naver's widget logic, resume after interruption, automatic cool-down on 403/429, reporter info, ten article reactions, commenter gender/age statistics, run report and completeness check. Selenium / ChromeDriver is no longer required.
- 2026/09/29 — Fixed a crash during ChromeDriver installation when two keywords (e.g. 5.18, 5·18) were run in parallel (superseded by the rewrite).
- 2026/08/23 — Added resume from where the run stopped.
- 2026/08/21 — Naver no longer provides UserIdNo, so the mark number is collected to distinguish comment authors.

## Contact

Bugs and suggestions: [hanjunleekr@hufs.ac.kr](mailto:hanjunleekr@hufs.ac.kr) or open an issue.

## Citation

If you use this code, please cite:

> Chae, Y. G., & Lee, H. J. (2026). Historical distortion and hate speech regarding the May 18 Democratic Uprising: A focus on the framing and thematic analysis of comments on portal news sites. *Korean Journal of Communication & Information, 138*, 194–239. https://www.kci.go.kr/kciportal/ci/sereArticleSearch/ciSereArtiView.kci?sereArticleSearchBean.artiId=ART003368454

```bibtex
@article{chae2026may18,
  author  = {Chae, Y. G. and Lee, H. J.},
  title   = {Historical Distortion and Hate Speech Regarding the May 18 Democratic Uprising: A Focus on the Framing and Thematic Analysis of Comments on Portal News Sites},
  journal = {Korean Journal of Communication \& Information},
  year    = {2026},
  volume  = {138},
  pages   = {194--239},
  url     = {https://www.kci.go.kr/kciportal/ci/sereArticleSearch/ciSereArtiView.kci?sereArticleSearchBean.artiId=ART003368454}
}
```

## Acknowledgements
This notebook was developed with the assistance of Claude (Anthropic).
