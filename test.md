Viewed test.md:22-37
Edited test.md
Ran command: `git push origin main`

Có, cách **"điền một lần dùng mãi mãi"** trên Linux là sử dụng Git Credential Helper. Bạn làm theo 2 bước sau:

### Bước 1: Bật trình lưu trữ thông tin đăng nhập
Chạy lệnh này để bảo Git lưu lại mật khẩu (Token) vào ổ cứng máy bạn:

```bash
git config --global credential.helper store
```

### Bước 2: Thực hiện Push và điền thông tin lần cuối
Bây giờ bạn chạy lệnh push như bình thường:

```bash
git push origin main
```

**Lúc này Git sẽ hỏi:**
1.  **Username:** Điền tên đăng nhập GitHub của bạn (ví dụ: `cuong111103hd`).
2.  **Password:** Dán mã **Personal Access Token (ghp_...)** mà bạn đã tạo vào đây.

---

### Kết quả:
*   Sau khi push thành công lần này, Git sẽ tạo một file ẩn tại `~/.git-credentials` để lưu Token đó.
*   Từ lần sau, khi bạn `push` hay `pull`, Git sẽ tự động lấy Token ra dùng mà không bao giờ hỏi lại bạn nữa.

**💡 Lưu ý:** Nếu trước đó bạn đã lỡ chạy lệnh `git remote set-url` có chứa token (như mình hướng dẫn ở trên), thì bạn nên đưa URL về dạng chuẩn để tránh lộ token trong file config của repo:
```bash
git remote set-url origin https://github.com/cuong111103hd/Day21-Track3-Finetuning-LLMs-LoRA-QLoRA.git
```
Sau đó mới chạy lệnh `store` ở Bước 1 thì sẽ bảo mật và chuyên nghiệp hơn!