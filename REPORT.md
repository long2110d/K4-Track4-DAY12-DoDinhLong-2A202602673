# REPORT — Lab Day 12

> All numerical results below come from the cited evidence files. Percentages in the final table are converted directly from stored rates.

| | |
|---|---|
| Họ và tên | Đỗ Đình Long |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | 2A202602673 |
| GitHub | long2110d |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | 1027 / 2222 |
| Số seed huấn luyện bạn dùng (`--seeds`) | 5 |

## Kết quả cuối

Source: `evidence/openloop.json`, `closedloop.json`, `gate.json`. Ranges are across training seeds; PercMLP+ L2 per-seed values are absent, so its mean and standard deviation are reported.

| Controller | Open-loop L2 (m) | Closed-loop log collisions (%) | Gate |
|---|---|---|---|
| constant-velocity | 1.2469592094421387 | 19.424210891432537 to 19.424210891432537 | Fail |
| IDM | 1.5204498767852783 | 0.0693721817551162 to 0.0693721817551162 | Baseline |
| EgoMLP | 0.7461634874343872 to 0.760897159576416 | 12.001387443635101 to 33.02115851543531 | Fail |
| PercMLP | 0.6123113632202148 to 0.6188545227050781 | 0.6243496357960457 to 1.5608740894901143 | Fail |
| PercMLP+ | Mean 0.8244664907455445; std 0.014708018741677549 (range unavailable) | 0.1387443635102324 to 0.2428026361429067 | Pass |

## Câu 1 · Dữ liệu
Bước 0 bạn phát hiện những vấn đề gì ở dữ liệu? Vì sao không dùng cột `Acc_LV`, và bạn thay bằng gì? Bao nhiêu quỹ đạo bị loại vì trùng, và vì sao chia theo quỹ đạo?

`evidence/data_audit.json` records 992679 rows, 9993 raw trajectories and 9610 clean trajectories after removing 383 exact duplicates.
Acc_LV correlates only 0.045353596290371 with the speed derivative, versus 0.9290867466753286 for follower acceleration; derive lead acceleration from smoothed Speed_LV instead.
Duplicates at distant IDs create leakage: deduplicate first, then split whole trajectories so the same motion cannot enter both training and evaluation.
Gap also needs the vehicle-length correction: the mean position-difference offset is 4.499999999999993 m; treating position difference as bumper gap overstates clearance.
The time step is 0.1 s, so adjacent frames are strongly dependent; splitting individual frames would give an optimistic estimate of generalization.

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

`evidence/replay.json` reports identical reruns on 600 segments; float32 changes bits on 600 segments but exceeds tolerance on 0.
Its maximum absolute difference is 0.0001222116138706042, with atol and rtol both 0.001: numerical representation changes need not imply meaningful behavior changes.
AEB changes 8 segments, including segment 175, with maximum difference 1.8407575407256296; selective low-TTC braking explains why only some segments should differ.
The evidence contains segment IDs but no frame-level trace for segment 175; the exact trigger frame and TTC cannot be verified from these files, so I do not claim to have inspected that trace.
Of 60364 frames, fastest has 0 late and 0 dropped, while realtime has 590 late, 581 dropped and 378 segments outside tolerance; fastest hides deadline failures and the effect of stale commands.

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

`evidence/scenarios.json` defines 2700 cells at cruise 14.0 m/s, gaps 2.7600510475215008 to 59.3123262996913 m and relative speeds 0.0 to 14.0 m/s.
Unavoidable fractions are 0.0362962962962963 and 0.10222222222222223 at immediate response and 0.6 s delay: approximately 3.63% and 10.22%, rounded to two decimal places.
IDM collision fractions equal those physical limits, increasing approximately 6.59 percentage points; its attributable_area stays 0.0, distinguishing unavoidable crashes from controller failures.
Full-brake validation reports 0 mismatches away from the frontier at both delays, supporting this interpretation rather than blaming IDM for every collision.
The human braking limit p99.9 is 3.6248373480294913 m/s squared, but 0.12666666666666668 of the grid requires more; 614 mined events do not establish coverage of those harder states.

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

| Controller | Mean L2 (m) | Mean collision fraction | Open / closed rank |
|---|---|---|---|
| constant-velocity | 1.2469592094421387 | 0.19424210891432536 | 3 / 3 |
| IDM | 1.5204498767852783 | 0.000693721817551162 | 4 / 1 |
| EgoMLP | 0.7522512078285217 | 0.21762053416579952 | 2 / 4 |
| PercMLP | 0.6146360754966735 | 0.00950398890045092 | 1 / 2 |
The following mean scores and ranks come from `evidence/closedloop.json` ranking and `evidence/openloop.json`; seed ranges are in the final-results table. Order flips are constant-velocity/IDM, constant-velocity/EgoMLP, IDM/EgoMLP and IDM/PercMLP. IDM controls clearance rather than copying human trajectories; EgoMLP lacks lead-vehicle information and accumulates errors under its own actions. The braking slice occupies only 0.04263350727212168 of open-loop windows, so average imitation error can hide rare safety-critical failures.

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

`evidence/closedloop.json` evaluates EgoMLP seeds 2222 through 2226 on 2883 trajectories; collision fractions range from 0.12001387443635102 to 0.3302115851543531, with mean 0.21762053416579952.
Seed 2226 has 346 crashes and Wilson ci95 [0.10865384397653853, 0.13238522100021666]; seed 2224 has 952 and ci95 [0.313280295305847, 0.3475947592866877].
Wilson expresses sampling uncertainty within a run under binomial assumptions; it does not cover training variability, domain shift or simulator bias, so both intervals and seed ranges matter.
Reporting only seed 2226 makes EgoMLP look safer than constant-velocity at 0.19424210891432536; reporting seed 2224 reverses that comparison, exposing the danger of selecting one run.
In `evidence/openloop.json`, EgoMLP L2 ranges only from 0.7461634874343872 to 0.760897159576416; stable imitation scores do not imply stable closed-loop safety.

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

| Controller | Replay any-collision fraction | Reactive any-collision fraction |
|---|---|---|
| constant-velocity | 0.8062962962962963 | 0.8062962962962963 |
| IDM | 0.8085185185185185 | 0.10222222222222223 |
| EgoMLP | 0.8418518518518519 | 0.8418518518518519 |
| PercMLP | 0.7670370370370371 | 0.39740740740740743 |
| PercMLP+ | 0.8066666666666666 | 0.11555555555555555 |
`evidence/agents.json`: T = 1.4 s gives W1 0.35102244386908976 versus 0.864079031368221 for the never-reacting agent; this improves distributional realism but does not validate every emergency response. Ego delay is 0.6 s. A replayed rear vehicle continues its old motion when IDM brakes, producing rear fraction 0.7062962962962963 versus reactive rear 0.0; these artificial crashes make IDM appear worse than PercMLP in replay, but reacting agents reverse that conclusion.

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

`evidence/gate.json` blocks PercMLP on R2, R3, R4 and R6: log upper bound 2.08%, braking-slice upper bound 7.9%, attributable grid area 29.52% and reactive-world area 39.7%.
Sparse hard-braking coverage and policy-induced distribution shift are plausible causes: `evidence/openloop.json` slice_share is 0.04263350727212168, while `scenarios.json` share_beyond_human_brake_limit is 0.12666666666666668; these support an explanation rather than independently proving causality.
The recorded fix in `evidence/fix.json` uses 20000 synthetic scenarios and 2 DAgger rounds, adding difficult states encountered by the policy instead of weakening gate thresholds.
PercMLP+ passes all rules: log upper bound 0.50%, slice upper bound 3.1%, attributable area 1.33% and reactive area 11.6%, versus IDM reactive area 10.2%; passing a non-inferiority margin does not establish superiority.
Accept release=true within the lab only. At held-out speed 20.0 m/s, PercMLP+ attributable area still reaches 0.017777777777777778 versus IDM 0.0; broader independent tests and supervised real-world validation should precede actual release.

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

Coverage is limited: `evidence/scenarios.json` has 2700 cells at cruise 14.0 m/s and 614 mined braking events; expand speeds, roads, weather and independently selected rare events before generalizing a release decision.
Agent realism is limited: `evidence/agents.json` calibrates T = 1.4 with W1 0.35102244386908976; matching headway distributions does not prove correct emergency reactions, so validate interacting drivers and multiple behavior models on real data.
Statistical support is limited: `evidence/closedloop.json` has 2883 held-out trajectories and 177 braking trajectories; dependent windows and rare outcomes require more independent contexts, seed runs and uncertainty estimates.
Sensor and timing coverage is limited: noise uses gap_std_m 1.0 and lead_speed_std_m_s 0.5, while `evidence/replay.json` already records 581 dropped frames; test correlated errors, outages, actual deadlines and hardware execution.
Generalization remains uncertain: `evidence/fix.json` tests 20.0 m/s but PercMLP+ still reaches attributable area 0.017777777777777778; use an untouched evaluation set, hardware testing and supervised field trials before extending the gate conclusion.
