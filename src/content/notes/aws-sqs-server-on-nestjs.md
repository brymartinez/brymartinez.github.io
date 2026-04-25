---
title: AWS SQS Server on NestJS
description: Notes on structuring an SQS-driven server with NestJS.
pubDate: 2026-04-25
topic: NestJS
draft: false
---

This processes controller entries with `@EventPattern({})` or `@EventPattern('')` for null entries.
```typescript
import { Logger } from '@nestjs/common';
import { CustomTransportStrategy, Server } from '@nestjs/microservices';
import { Context, SQSEvent, SQSRecord } from 'aws-lambda';

export class AWSSQSServer extends Server implements CustomTransportStrategy {
  event: SQSEvent;
  context: Context;
  constructor(event: SQSEvent, context: any) {
    super();
    this.event = event;
    this.context = context;
  }

  /**
   * This method is triggered when you run "app.listen()".
   */
  async listen(callback: () => void) {
    const records = this.getRecords();
    for await (const record of records) {
      const key = this.getMessageAttribute(record);
      const msg = this.getMessage(record);
      Logger.debug(key, 'SQSKey');
      Logger.debug(msg, 'SQSMessage');
      Logger.debug(record, 'SQSRecord');
      Logger.log([...this.messageHandlers.keys()]);
      const handler = this.messageHandlers.get(key);
      if (handler) {
        await handler(msg, this.context);
      } else {
        Logger.error(`${key} is not a valid handler.`);
      }
    }
    callback();
  }

  /**
   * This method is triggered on application shutdown.
   */
  close() {
    return;
  }

  /**
   * This method converts all messageAttributes to a string with format `key:value`.
   * Example:
   * MessageAttributeMap: `[{ key2: value1, key2: value2 }]`
   * Output: [{ 0: key1:value1, 1: key2:value2}]
   * @private
   * @param {SQSRecord} record
   * @return {*}  {string}
   * @memberof AWSSQSServer
   */
  private getMessageAttribute(record: SQSRecord): string {
    const messageAttributeMap = record.messageAttributes;
    const returnArray: string[] = [];
    Object.keys(messageAttributeMap).forEach((key) => {
      returnArray.push(`${key}:${messageAttributeMap[key].stringValue}`);
    });
    const stringifiedKey = JSON.stringify(Object.assign({}, returnArray));
    return stringifiedKey === '{}' ? '' : stringifiedKey;
  }

  /**
   * This returns the Message part of SQS
   */
  private getMessage(record: SQSRecord): any {
    return record.body;
  }

  private getRecords(): SQSRecord[] {
    return this.event.Records;
  }
}
```