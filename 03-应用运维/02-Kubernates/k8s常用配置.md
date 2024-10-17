保活机制
在滚动重启时，会保证部分pod不会立即停止，在重启的pod健康检查通过后，才会使保活的pod重启

          livenessProbe:
            tcpSocket:
              port: 8091
            initialDelaySeconds: 60
            periodSeconds: 10
            timeoutSeconds: 10
            failureThreshold: 3
          readinessProbe:
            tcpSocket:
              port: 8091
            initialDelaySeconds: 60
            periodSeconds: 10