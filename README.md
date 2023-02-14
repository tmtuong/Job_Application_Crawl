# MD_ApplicationCrawl

## Mục tiêu

Đây là project để lấy thông tin ứng viên nộp CV trên 3 web: Glints, CareerBuilder, TopCV. Sau đó lưu thông tin lên Google Sheets và lưu file CV .pdf lên drive của công ty để lưu trữ.

## Tóm tắt các file

     - client_secret.json: Thông tin về Google Drive API dùng để redirect tới Drive để upload CV.
     - settings.yaml: Một số settings cần thiết lập để mở Google Drive từ máy tính. Chỉ thay đổi client_id và client_secret (cả 2 lấy từ client_secret.json) để chạy.
     - login.json: Thông tin về tài khoản và mật khẩu để đăng nhập vào CareerBuilder.
     - CareerBuilder_Crawl.ipynb: Lấy về thông tin ứng viên trên CareerBuilder.
     - Glints_Crawl.ipynb: Lấy về thông tin ứng viên trên Glints.
     - TopCV.ipynb: Lấy về thông tin ứng viên trên TopCV.
     - GetCV.ipynb: Lấy về thông tin ứng viên trên cả 3 nền tảng kể trên.
     
## Idea

Để lấy được thông tin ứng viên đầu tiên cần phải đăng nhập vào trang web dành cho nhà tuyển dụng. Có 2 cách để đăng nhập vào để lấy thông tin là:

     - Đối với web có API: Sử dụng requests và truyền headers là authorization vào.
     - Đối với web không có API: Dùng selenium để đăng nhập.
     
Sau khi đăng nhập, tùy vào mỗi trang web mà sẽ có những cách lấy ra data cần thiết khác nhau. Cụ thể từng web sẽ được trình bày ở các phần sau.

Các file CV tải về toàn bộ sẽ được lưu trữ trên Drive và được đặt tên có dạng <tên ứng viên>\_<vị trí apply>(<ngày apply>).
     
## CareerBuilder_Crawl.ipynb

Trang web https://careerbuilder.vn/vi/employers yêu cầu đăng nhập và không cung cấp API nên dùng **Selenium để đăng nhập và cào bề mặt**.

### Các bước thực hiện

Đầu tiên cần gọi truy cập và sheet hiện đang lưu thông tin và lấy ra những cv hiện đã có, để loại trừ ra sau khi lấy dữ liệu về -> đảm bảo dữ liệu không trùng lặp.

Tiếp theo, thiết lập Options cho Chrome driver (quan trọng nhất là download.default_directory -  thư mục mặc định cho file tải về và download.prompt_for_download - hỏi trước khi tải về). Và mở Chrome driver (lưu ý phải có Chrome driver.exe trong máy) -> Truy cập trang web https://careerbuilder.vn/vi/employers/hrcentral/manageresume

![image](https://user-images.githubusercontent.com/124959517/218029216-6f6c7853-7959-48df-80c9-86970890e2d6.png)

Login bằng tài khoản công ty bằng cách tìm đến các ô nhập tài khoản, mật khẩu bằng find_element sau đó tự động điền giá trị từ file login.json bằng phương thức send_keys. Giữa các bước nên có khoảng dừng để không bị phát hiện là bot (dễ gây ra lỗi/ bị chặn).

Sau đó tìm đến hộp thoại chuyển đổi giữa các công việc đang tuyển dụng bằng cách find_element By.XPATH (để tìm XPATH bấm F12 là tìm vị trí của element sau đó Copy XPATH). Bấm vào từng công việc bắt đầu từ công việc thứ 2 đến hết, sau đó mới quay lại công việc đầu tiên (vì mặc định sẽ chọn là công việc đầu tiên nên nếu click bắt đầu từ công việc đầu tiên sẽ không thay đổi đường link -> không lấy được id của job từ link)

![image](https://user-images.githubusercontent.com/124959517/218031472-ba1c8e68-7e4a-4edd-a706-64766f4e7874.png)

Tách id của job từ các đường link đã lấy được sau đó thay vào đường link khác (đã có các params để lọc theo ngày - vì không cần lấy hết tất cả CV). Đừng link để lấy CV mỗi job có dạng **https://careerbuilder.vn/vi/employers/hrcentral/manageresume/1/ + id +/*/0/0/03-02-2023/*/7/2/6/2/0/desc/hr.1661228713/1**.

Lặp qua từng đường link trên để lấy về đường link dẫn tới thông tin của mỗi ứng viên. Mỗi trang chỉ limit 20 ứng viên nên trước tiên cần xác định số ứng viên trả về, nếu lớn hơn 20, cần có thêm thao tác bấm qua trang.

So sánh với các ứng viên đã lưu trên Sheet bằng id (mỗi lần nộp sẽ có 1 id khác nhau), nếu id đã có sẽ không lấy về nữa.

Lặp qua các đường link ứng viên còn lại để lấy về thông tin ứng viên. Sau đó tải CV về bằng đường link https://careerbuilder.vn/vi/employers/popup/downloadresume?resume_id={candidate_id}, cv tải về có dạng <Tên ứng viên>\_<mã apply>. Nên sau khi tải về cần quét lại folder đã tải về và lấy về tên đã tải về, nếu đuôi <mã apply> khớp với mã apply hiện tại trong dataframe thì sẽ đổi tên file theo đúng format mong muốn.

Cuối cùng, upload file đã đổi tên lên Google Drive thông qua API và quét hết toàn bộ file trong folder trên drive nếu match với tên file được lưu trong dataframe thì lấy link của file đó và lưu vào dataframe.

### Một số lưu ý
 
 - Sẽ phải Authorize thủ công lần đầu khi kết nối máy tính với Drive API.

 - File sau khi tải về nên xóa cho nhẹ máy. Vì đã upload lên drive nên không cần lưu trong máy nữa.

 - Vì CV sẽ được tải xuống sau đó mới đổi tên và upload lên Drive nên code đôi khi sẽ bị lỗi do tốc độ tải xuống chậm -> Không quét được file để đổi tên. Để hạn chế trường hợp này đã cho code ngưng 30s sau khi tải để đảm bảo tải xuống hoàn tất trước khi chạy cell tiếp theo.

 - CareerBuilder không chia theo tên công ty nên tên công ty sẽ được chia theo HR quy định (thêm tay nếu có tin tuyển dụng mới).

   ![image](https://user-images.githubusercontent.com/124959517/218037873-12a98eb9-8079-42fa-b6a5-2ee33c60f963.png)
   
## Glints_Crawl.ipynb

 Trang web https://employers.glints.vn/ có api nên sẽ sử dụng **requests để requests API link**. 
 Link API: 
  - Job API: https://employers.glints.vn/api/companies/{company_id}/jobs?include=jobSalaries%2C+Groups%2C+City%2C+Country&isPublic=true&includeJobStatusBreakdown=true&includeApplicationStatusBreakdown=true&order=updatedAt+DESC&limit=100&offset=0&where=%7B%22status%22%3A%5B%22OPEN%22%2C%22IN_REVIEW%22%5D%7D&includeViewCount=true&includeExpiryReason=true. Với tham số sẽ thay đổi là {company_id}.
  - Candidate API: https://employers.glints.vn/api/jobs/{job_id}/applications?where=%7B%22JobId%22%3A%2201a93366-a25e-495f-a77a-a19a35b1a9ad%22%2C%22status%22%3A%22NEW%22%7D&order=createdAt+DESC&includeFollowUpRequest=true&includeStatusBreakdown=true&includeApplication=true&newContractWithBreakdown=true&limit=1000&offset=0. Với tham số thay đổi là {job_id}.

### Các bước cần thực hiện

Đầu tiên, cần đăng nhập vào web và lấy về authorization headers thủ công. F12 -> Chọn network -> Bấm bất kì đường link nào -> Chọn headers.
![image](https://user-images.githubusercontent.com/124959517/218041613-8e8423da-80a3-45d6-9b19-df84786e7b70.png)

Lưu thành biến headers để requests. Câu lệnh để requests sẽ là requests.get(url, headers = headers)
![image](https://user-images.githubusercontent.com/124959517/218042137-9fbf000a-08e4-4545-9dab-15430c319a01.png)

Requests link Job API để lấy ra id của các tin tuyển dụng hiện đang **active** (job không active thì xem và sửa đổi lại params chỗ link). Vì chỉ có 2 công ty là Genkin với Maido nên company_id sẽ được lấy thủ công và lưu dưới dạng dictionary.
![image](https://user-images.githubusercontent.com/124959517/218046133-3e61d5dd-0f56-4941-bb5f-2098a0f20f3c.png)

Lặp qua mỗi job_id đã lấy về được và thay vào link Candidate API để lấy thông tin ứng viên và lưu vào dataframe. Sau đó Take note ứng viên đã tốt nghiệp hay chưa dựa trên start_edu và end_edu lấy về được. Có 3 kết quả trả về cho cột graduated là:
 - Yes: Cả 2 thông tin đều có đầy đủ và end_date < today.
 - No: Cả 2 thông tin đều có đầy đủ và end_date > today. Trong trường hợp này sẽ take note là còn **"{month(end_date) - month(today)} nữa tốt nghiệp"**.
 - Unknown: Thiếu 1 trong 2 trường start_edu, end_edu hoặc cả 2.

Để tải về cv, requests đường link https://s3-ap-southeast-1.amazonaws.com/glints-dashboard/resume/{cv_id} (với tham số truyền vào là cv_id đã lấy được ở bước trên) và lấy ra content sau đó mở file với đường dẫn là <vị trí tải về mong muốn>/<tên file> và ghi lại content đã lấy về vào.

Cuối cùng, upload file đã đổi tên lên Google Drive thông qua API và quét hết toàn bộ file trong folder trên drive nếu match với tên file được lưu trong dataframe thì lấy link của file đó và lưu vào dataframe.

### Một số lưu ý

 - Sẽ phải Authorize thủ công lần đầu khi kết nối máy tính với Drive API.

 - Headers authorization sau một thời gian sẽ hết hạn, phải tự lấy thủ công.

 - Không thể đăng nhập bằng selenium để lấy API.

 - Cách ghi content để save file pdf chỉ có thể thực hiện khi đường dẫn web của file có kết thúc là .pdf.

## TopCV.ipynb

Trang web https://tuyendung.topcv.vn/ có api nên sẽ sử dụng **requests để requests API link**.

Link API: https://tuyendung-api.topcv.vn/api/v1/cv-management/cvs?page={page}. API này chứa toàn bộ thông tin cần thiết nên không cần phải sử dụng nhiều API link.

## Các bước cần thực hiện:

Đầu tiên, cần đăng nhập vào web và lấy về authorization headers thủ công. F12 -> Chọn network -> Bấm bất kì đường link nào -> Chọn headers.
![image](https://user-images.githubusercontent.com/124959517/218041613-8e8423da-80a3-45d6-9b19-df84786e7b70.png)

Lưu thành biến headers để requests. Câu lệnh để requests sẽ là requests.get(url, headers = headers)
![image](https://user-images.githubusercontent.com/124959517/218042137-9fbf000a-08e4-4545-9dab-15430c319a01.png)

Tạo vòng lặp while để tăng số page và thay vào link API để lấy về thông tin ứng viên. Vòng lặp sẽ kết thúc khi requests không ra dữ liệu (độ dài dữ liệu requests về bằng 0) - hết page.

Để tải về cv, requests đường link để tải cv (đã lấy được ở bước trên) và lấy ra content sau đó mở file với đường dẫn là <vị trí tải về mong muốn>/<tên file> và ghi lại content đã lấy về vào.

Cuối cùng, upload file đã đổi tên lên Google Drive thông qua API và quét hết toàn bộ file trong folder trên drive nếu match với tên file được lưu trong dataframe thì lấy link của file đó và lưu vào dataframe.

### Một số lưu ý

 - Sẽ phải Authorize thủ công lần đầu khi kết nối máy tính với Drive API.

 - Headers authorization sau một thời gian sẽ hết hạn, phải tự lấy thủ công.

 - Không thể đăng nhập bằng selenium để lấy API.

 - Cách ghi content để save file pdf chỉ có thể thực hiện khi đường dẫn web của file có kết thúc là .pdf.

## GetCV.ipynb

File này là tổng hợp của tất cả các file trên được gọi ra để chạy bằng lệnh magic (%run <file>)
![image](https://user-images.githubusercontent.com/124959517/218062973-1538a595-3de4-409a-a098-a45e50abc8d6.png)

Mỗi file khi chạy đều có một bộ đếm lỗi. Nếu chạy lần đầu lỗi thì sẽ chạy lại lần 2 sau 5 phút. Nếu lần 2 lỗi thì mới raise lên để sửa.
![image](https://user-images.githubusercontent.com/124959517/218064187-4c44d87e-91eb-415a-9298-10a7087f8e56.png)
![image](https://user-images.githubusercontent.com/124959517/218064229-6fda6fee-39bf-45d7-ad8e-98c628c1a67d.png)

File này sẽ được hẹn giờ chạy vào lúc 8h sáng hằng ngày.
