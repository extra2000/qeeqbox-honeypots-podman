FROM localhost/qeeqbox/honeypots

RUN apt update && apt install dos2unix

COPY entrypoint.sh /bin/entrypoint.sh
COPY service/run.sh /usr/local/bin/exec-honeypot
RUN dos2unix /bin/entrypoint.sh && chmod a+x /bin/entrypoint.sh
RUN dos2unix /usr/local/bin/exec-honeypot && chmod a+x /usr/local/bin/exec-honeypot

ENTRYPOINT ["/bin/entrypoint.sh"]
CMD ["/usr/local/bin/exec-honeypot"]
