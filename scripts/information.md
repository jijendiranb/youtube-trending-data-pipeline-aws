Bronze Bucket Name - yt-data-bronze-layer
Silver Bucket Name - yt-data-silver-layer
Gold Bucket Name -  yt-data-gold-layer

Script Bucket - yt-data-scripts-list
Athena Bucket -yt-athena-results-layer

SNS ARN - arn:aws:sns:ap-south-1:087380083416:yt-data-messenger:048a9c8d-be32-417f-af8b-e9acb07623d7

Glue Bronze - yt-data-bronze
Glue Silver - yt-data-silver
Glue Gold -  yt-data-gold


--bronze_database  yt-data-bronze
--bronze_table raw_statistics
--silver_bucket  yt-data-silver-layer
--silver_database  yt-data-silver
--silver_table clean_statistics

--silver_database  yt-data-silver
--gold_bucket yt-data-gold-layer
--gold_database yt-data-gold