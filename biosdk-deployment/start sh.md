#!/bin/bash

echo "Waiting for License Daemon to become ready..."

while true; do
  OUTPUT=$(/usr/sbin/Multiprotect/Multiprotect_Manager 2>&1)

  echo "$OUTPUT"

  # Check if daemon is STILL NOT READY
  if echo "$OUTPUT" | grep -q "The Protection Service is not available"; then
    echo "Daemon not ready yet..."
    sleep 2
  else
    echo "Daemon is READY ✅"
    break
  fi
done

echo "Starting Java app..."

exec java \
-Dloader.path=/home/Biosdk/idemiaSdk-1.1.9-SNAPSHOT-jar-with-dependencies.jar \
-Dbiosdk_class=com.idemia.sdk.IdemiaEngine \
-Dspring.cloud.config.label="${spring_config_label_env}" \
-Dspring.profiles.active="${active_profile_env}" \
-Dspring.cloud.config.uri="${spring_config_url_env}" \
-jar /home/Biosdk/biosdk-services-1.1.3.jar
