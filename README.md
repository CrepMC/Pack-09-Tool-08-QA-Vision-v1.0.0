# Pack 09 — Tool 08: QA Vision

**Version:** 1.0.0  
**Upstream:** Generated Video + T07 CinematicPlan + T06 EnvironmentPlan + T05 MotionPlan + T04 CameraPlan + T03 CreativeBrief + T02 ProductDNA  
**Downstream:** T09 Prompt Optimizer  
**Primary artifact:** `QAReport`

## Mục tiêu

QA Vision là cổng kiểm định sau generation. Nó không tạo video, không sửa video, không sửa prompt và không “tự cho qua” lỗi.

Tool này chịu trách nhiệm:
- đọc generated video;
- đọc mọi upstream contract;
- kiểm tra output theo Product / Camera / Motion / Environment / Cinematic;
- ghi issue có timecode, severity, evidence;
- phân biệt blocking vs non-blocking;
- tính score theo từng domain;
- quyết định PASS / FAIL / NEEDS_REVIEW;
- tạo correction targets có cấu trúc cho T09 Prompt Optimizer;
- preserve những phần đã pass để T09 sửa tối thiểu.

## Các lỗi QA phải bắt

### Product
- neckline/collar thay đổi;
- logo/pattern đổi;
- nút/túi/dây rút biến mất;
- accessory biến mất hoặc biến dạng;
- màu sản phẩm drift;
- material đổi;
- sản phẩm bị crop/che.

### Camera
- output sai 9:16;
- camera tĩnh khi cần tracking;
- zoom thay dolly;
- start/end framing bị đảo;
- crop chân;
- orbit/pan ngẫu nhiên;
- subject scale drift;
- angle jump/cut ngoài kế hoạch.

### Motion
- tay chạm cổ áo;
- hand-in-pocket khi cấm;
- teleport/foot sliding;
- turn quá mức;
- over-choreography;
- cuối video mid-step;
- final body state chưa settled.

### Environment
- background bị thay;
- thêm người/props;
- sign/text đổi;
- anchor biến mất;
- wall/floor morph;
- parallax sai;
- subject đi xuyên vật thể.

### Cinematic
- màu sản phẩm drift;
- vải matte thành glossy;
- skin plastic;
- DOF làm mờ sản phẩm;
- motion blur smear;
- bloom/haze quá mạnh;
- highlight clipping;
- shadow crush;
- environment anchors mất readability.

## Quy tắc cốt lõi

1. QA phải evidence-based.
2. Không PASS chỉ vì “nhìn tổng thể đẹp”.
3. Product fidelity priority cao nhất.
4. Hard requirement fail → toàn video FAIL dù style score cao.
5. QA không rewrite prompt. Nó chỉ tạo correction targets.
6. QA không regenerate.
7. QA phải preserve passing domains để T09 không sửa quá tay.
8. Issue cần timecode/range nếu có thể.
9. Nếu evidence không đủ, dùng NEEDS_REVIEW/UNKNOWN thay vì invent.
10. Score không được che mất blocking failure.

## Ba prompt chính

1. [`Prompt-01-Foundation-Video-Ingest-QA-Contract-and-UI.md`](./prompts/Prompt-01-Foundation-Video-Ingest-QA-Contract-and-UI.md)
2. [`Prompt-02-Multidomain-Vision-Review-Scoring-Evidence-and-Timecode-Engine.md`](./prompts/Prompt-02-Multidomain-Vision-Review-Scoring-Evidence-and-Timecode-Engine.md)
3. [`Prompt-03-Production-Hardening-T09-Handoff-Validation-and-Release.md`](./prompts/Prompt-03-Production-Hardening-T09-Handoff-Validation-and-Release.md)

## Definition of Done

- import video + all contracts;
- QA domains separated;
- timecoded issue reports;
- evidence references;
- severity/blocking rules;
- pass/fail logic;
- domain scores;
- product consistency checks;
- camera/motion/environment/cinematic checks;
- correction targets for T09;
- no prompt rewriting;
- no regeneration;
- T08→T09 handoff valid;
- build/typecheck/tests pass.
