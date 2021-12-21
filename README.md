# How to upload data to Data Lake with GCP

## Data Lake 

เป็นที่เก็บข้อมูลขนาดใหญ่ รองรับข้อมูลทุกรูปแบบที่เป็นไฟล์
คล้าย Harddrive ในเครื่องคอมพิวเตอร์ของเรา แต่มีระบบป้องกันข้อมูลสูญหายที่ดีกว่า
เก็บได้ทั้ง Structured Data, Semi Structured Data และ Unstructured Data
Data Lake บางครั้งก็ถูกเรียกว่า object storage หรือ blob storage

### วิธีเลือก Data Storage: เลือกตามจุดประสงค์การใช้งาน

1. เก็บข้อมูลในระบบที่มีการใช้งานและเปลี่ยนแปลงตลอดเวลา เช่น
  - ฐานข้อมูลที่ใช้งานในธุรกิจ หรือ อุตสาหกรรม
  - การใช้งานร่วมกับ application หรือเว็บไซต์
    - ใช้ Database (DB)

2. เก็บข้อมูลเพื่อการวิเคราะห์ข้อมูล สำหรับข้อมูลที่ไม่มีการเปลี่ยนแปลงแล้ว มักจะดึงข้อมูลจาก Database มาเก็บ เช่น 
  - การนำข้อมูลไปวิเคราะห์หรือสร้างโมเดล
  - ใช้ข้อมูลเพื่อสร้าง BI dashboard / Visualization
    - ใช้ Data Warehouse (DW)

3. เก็บข้อมูล Unstructured Data เช่น รูปภาพ, ไฟล์เสียง, วีดิโอ หรือ การเก็บข้อมูลดิบ(raw data) ในรูปแบบของไฟล์เช่น .txt, .csv ในราคาประหยัด
     - Data Lake (DL)


### วิธี Upload ไฟล์ข้อมูลเข้า Google Cloud Storage ที่เราจะใช้เป็น Data Lake
Prerequisite:
1. สร้าง Project on Google Cloud
2. สร้าง Bucket เก็บข้อมูล

gsutil คำสั่งในการใช้งานร่วมกับ Google Cloud
gsutil เป็น command ที่มาพร้อมกับ Cloud SDK ที่ต้อง install เพิ่ม ถ้าต้องการใช้งานผ่าน command line บน local หรือสามารถใช้งานผ่าน Cloud Shell ได้ทันที

การอ้างอิง path ใน Google Cloud Storage จะต้องขึ้นต้นด้วย gs:// เสมอ

- Documentation การใช้งาน gsutil: https://cloud.google.com/storage/docs/how-to
- gsutil Cheatsheet: https://alexisperrier.com/gcp/2017/11/02/gsutil-cheatsheet.html
- Install Cloud SDK: https://cloud.google.com/sdk/docs/install


วิธี Upload Data เข้า Google Cloud Storage

 
### Google Cloud Storage (GCS)

1. สร้าง Bucket สามารถได้หลายวิธีดังนี้
- Cloud Console บนเว็บ UI
- gsutil command ผ่าน Cloud Shell

2. อัปโหลดข้อมูล สามารถทำได้หลายวิธีดังนี้
- อัปโหลดผ่าน Cloud Console บนเว็บ UI
- ใช้คำสั่ง gsutil ผ่าน Cloud Shell
- ใช้โค้ด Python ผ่าน Python SDK library

  - Upload : https://cloud.google.com/storage/docs/samples/storage-upload-file#storage_upload_file-python
  - Download : https://cloud.google.com/storage/docs/downloading-objects#storage-download-object-python


### Code python upload data to Google Cloud Storage

```python
from google.cloud import storage


def download_blob(bucket_name, source_blob_name, destination_file_name):
    """Downloads a blob from the bucket."""
    # The ID of your GCS bucket
    # bucket_name = "your-bucket-name"

    # The ID of your GCS object
    # source_blob_name = "storage-object-name"

    # The path to which the file should be downloaded
    # destination_file_name = "local/path/to/file"

    storage_client = storage.Client()

    bucket = storage_client.bucket(bucket_name)

    # Construct a client side representation of a blob.
    # Note `Bucket.blob` differs from `Bucket.get_blob` as it doesn't retrieve
    # any content from Google Cloud Storage. As we don't need additional data,
    # using `Bucket.blob` is preferred here.
    blob = bucket.blob(source_blob_name)
    blob.download_to_filename(destination_file_name)

    print(
        "Downloaded storage object {} from bucket {} to local file {}.".format(
            source_blob_name, bucket_name, destination_file_name
        )
    )


def upload_blob(bucket_name, source_file_name, destination_blob_name):
    """Uploads a file to the bucket."""
    # The ID of your GCS bucket
    # bucket_name = "your-bucket-name"
    # The path to your file to upload
    # source_file_name = "local/path/to/file"
    # The ID of your GCS object
    # destination_blob_name = "storage-object-name"

    storage_client = storage.Client()
    bucket = storage_client.bucket(bucket_name)
    blob = bucket.blob(destination_blob_name)

    blob.upload_from_filename(source_file_name)

    print(
        "File {} uploaded to {}.".format(
            source_file_name, destination_blob_name
        )
    )


if __name__ == "__main__":
    upload = input("Upload (u) or Download (d)?")
    file_name = input("What's local name to : ")
    gcs_file_name = input("What's gcs file name: ")

    bucket_name = "datalake-booksales-using-gsutil"

    if upload.lower() == "upload" or upload.lower() == "u":
        upload_blob(
            bucket_name=bucket_name,
            source_file_name=file_name,
            destination_blob_name=gcs_file_name
        )
    elif upload.lower() == "download" or upload.lower() == "d":
        download_blob(
            bucket_name=bucket_name,
            source_blob_name=gcs_file_name,
            destination_file_name=file_name
        )
    else:
        print("Please input upload (u) or download (d)")
 
```
