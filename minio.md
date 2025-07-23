





    # values.yaml
    minio:
      enabled: true
      # You can customize MinIO settings here if needed, e.g., persistence
      # persistence:
      #   size: 10Gi

    loki:
      config:
        chunk_store_config:
          max_look_back_period: 24h
        schema_config:
          configs:
            - from: 2020-10-24
              store: boltdb-shipper
              object_store: s3
              schema: v11
              index:
                prefix: index_
                period: 24h
        storage_config:
          boltdb_shipper:
            active_index_directory: /var/loki/index
            cache_location: /var/loki/cache
            resync_interval: 5m
            shared_store: s3
          aws:
            s3:
              endpoint: minio.default.svc.cluster.local:9000 # Replace 'default' with your namespace if different
              bucketnames: loki-chunks
              accesskeyid: admin # Replace with your MinIO access key
              secretaccesskey: admin123 # Replace with your MinIO secret key
              s3forcepathstyle: true


