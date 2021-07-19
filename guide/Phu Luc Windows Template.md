
Cài đặt template cần thực hiện những việc sau:

------------------------------------------
* Installed virtio driver
* Registry clock RealTimeIsUniversal=0
* Set timezone to UTC+7
* Disable IESC
* Turn off autoplay on all drives
* Disable shutdown event tracker
* Turn on Remote Desktop (but not open windows firewall yet)
* Installed 7-zip 9.60
* Disable following services (set to manual startup):
  * Print spooler
  * Server (files and printer sharing)
* Windows Update to latest important
* Config windows update to only check, not download automatically
* Installed Firefox
* Installed Noted Pad ++ 
* Trường hợp cài đặt Virtio Balloon Driver báo chấm than, cài đặt lại bản mới nhất và uninstall driver + tích bỏ check và reload lại driver, máy sẽ tự động cập nhập lại Driver Virtio Balloon