create-buckets:
	awslocal s3 mb s3://localstack-thumbnails-app-images
	awslocal s3 mb s3://localstack-thumbnails-app-resized

see-buckets:
	awslocal s3 ls

# add bucket names into the parameter store
create-ssm-parameters:
	awslocal ssm put-parameter \
		--name "/localstack-thumbnail-app/buckets/images" \
		--type "String" \
		--value "localstack-thumbnails-app-images"
	awslocal ssm put-parameter \
		--name "/localstack-thumbnail-app/buckets/resized" \
		--type "String" \
		--value "localstack-thumbnails-app-resized"

# get parameters by path
# if we don't use the recursive flag, we need to specify the parameter name
# it will return empty if we don't use the recursive flag
get-ssm-parameters:
	awslocal ssm get-parameters-by-path \
		--path "/localstack-thumbnail-app/" \
		--recursive


create-sns-dlq-topic:
	awslocal sns create-topic \
		--name failed-resize-topic

sub-email-address:
	awslocal sns subscribe \
		--topic-arn arn:aws:sns:us-east-1:000000000000:failed-resize-topic \
		--protocol email \
		--notification-endpoint test@test.com


