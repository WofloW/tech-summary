# Commands

Use AWS cli to access glacier

[steps](https://softwaredevelopmentstuff.com/2017/05/02/downloading-an-aws-glacier-archive-step-by-step/)

## Get inventory information
aws glacier initiate-job --account-id - --vault-name photos --job-parameters '{"Type": "inventory-retrieval"}'
aws glacier get-job-output --account-id - --vault-name photos --job-id <previous-job-id> output.json


## Download inventory file

### Create request.json
```json
{
  "Type": "archive-retrieval",
  "ArchiveId": "<archive-id>",
  "Description": "Download archive"
}
```

### Initiating the download archive request
It may cost hours to finish the request.
aws glacier initiate-job \
    --account-id - \
    --vault-name photos \
    --job-parameters file://request.json


### Check the status
aws glacier describe-job \
    --account-id - \
    --vault-name photos \
    --job-id <job-id>

Once the completed is true, you can go to the next step.

### Downloading the archive
aws glacier get-job-output \
    --account-id - \
    --vault-name photos \
    --range bytes=0-180824264 \
    --job-id <job-id2> \
    photos.part1

### Split and combine
If the range is too big, split the file and combine the files
cat photos.part1 \
    photos.part2 \
    photos.part3 \
    photos.part4 \
    photos.part5 > photos.zip



