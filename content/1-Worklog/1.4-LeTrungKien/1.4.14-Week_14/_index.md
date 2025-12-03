+++
title = "Week 14 Worklog"
weight = 14
chapter = false
pre = " <b> 1.4.14. </b> "
+++

## Week 14 Objectives


## Tasks to be carried out this week
| Day |                                Task                                                   | Start Date | Completion Date |                   Reference Material                     |
|-----|---------------------------------------------------------------------------------------|------------|-----------------|--------------------------------------------------------- |
| 1   ||08/11/2025|08/11/2025||
| 2   ||09/11/2025|09/11/2025||
| 3   ||10/11/2025|10/11/2025||
| 4   ||11/11/2025|11/11/2025||
| 5   ||12/11/2025|12/11/2025||



# Đường dẫn tới thư mục của Dương Bình Minh
$targetDir = "content/1-Worklog/1.3-DuongBinhMinh"

# Kiểm tra xem thư mục có tồn tại không
if (-not (Test-Path $targetDir)) {
    Write-Host "Lỗi: Không tìm thấy thư mục $targetDir" -ForegroundColor Red
    break
}

$folders = Get-ChildItem -Path $targetDir -Directory

foreach ($folder in $folders) {
    if ($folder.Name -match "Week_(\d+)") {
        $weekNum = $matches[1]
        
        # Định nghĩa 2 đường dẫn file có thể xảy ra
        $correctFile = Join-Path $folder.FullName "_index.md"
        $wrongFile = Join-Path $folder.FullName "index.md"

        # BƯỚC 1: TỰ ĐỘNG SỬA TÊN FILE (Nếu đang là index.md thì đổi thành _index.md)
        if ((Test-Path $wrongFile) -and (-not (Test-Path $correctFile))) {
            Rename-Item -Path $wrongFile -NewName "_index.md"
            Write-Host "-> Đã sửa tên file cho Week $weekNum (index.md => _index.md)" -ForegroundColor Magenta
        }

        # BƯỚC 2: SỬA NỘI DUNG FILE _index.md
        if (Test-Path $correctFile) {
            # Đọc nội dung file
            $content = Get-Content -Path $correctFile -Raw -Encoding UTF8

            # Tạo nội dung pre mới
            $newPreLine = "pre = `" <b> 1.3.$weekNum. </b> `""

            # Kiểm tra xem file đã có dòng pre chưa
            if ($content -match '(?m)^pre\s*=') {
                # Nếu có rồi -> Thay thế
                $newContent = $content -replace '(?m)^pre\s*=\s*".*"', $newPreLine
            } else {
                # Nếu chưa có -> Thêm mới vào sau dòng weight
                # (Phòng trường hợp file week 13, 14 cũ chưa có dòng pre)
                $newContent = $content -replace '(?m)^weight\s*=\s*(\d+)', "weight = `$1`n$newPreLine"
            }

            # Ghi đè lại file
            Set-Content -Path $correctFile -Value $newContent -Encoding UTF8
            
            Write-Host "Đã sửa Week $($weekNum): $newPreLine" -ForegroundColor Green
        } else {
            Write-Host "Bỏ qua $folder.Name (Không tìm thấy file markdown nào)" -ForegroundColor DarkGray
        }
    }
}

Write-Host "Hoàn tất cập nhật!" -ForegroundColor Cyan