# Automated S3 Bucket Cleanup Using AWS Lambda and Boto3

# Automate the deletion of files older than 30 days in a specific S3 bucket.

# S3 Setup:
In the AWS Console, go to S3 → Buckets → Create bucket

Created the bucket in the same region as where my Lambda function is running.
<img width="777" height="292" alt="image" src="https://github.com/user-attachments/assets/f4dfeb54-42d5-4b96-96e7-86df1becda52" />

# Upload Files:
Upload multiple files to the bucket (e.g., text or image files).
<img width="1242" height="663" alt="image" src="https://github.com/user-attachments/assets/2394eca3-9c72-4e2a-8bae-34c21d55e5f2" />

To simulate “old files,” you can:
Temporarily change your system date and upload files -  I have tried this option but this is not working, so i have tried the below option to change the metadata with below commands

aws s3 cp C:\Users\durga\Downloads\notes2.txt s3://durga-s3-cleanup-bucket/ --metadata last-modified=2025-09-15T10:30:00Z

aws s3 cp "C:/Users/durga/Downloads/image1.jpg" s3://durga-s3-cleanup-bucket/ --metadata last-modified=2025-09-15T10:30:00Z

Upload some files and then manually edit their metadata (though AWS S3 records the true LastModified timestamp). Example one image is shown here.

<img width="1375" height="257" alt="image" src="https://github.com/user-attachments/assets/23c23918-925a-402f-ba09-90dd9895a046" />

