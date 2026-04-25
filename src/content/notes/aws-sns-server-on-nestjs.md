---
title: AWS SNS Server on NestJS
description: Notes on structuring an SNS-driven server with NestJS.
pubDate: 2026-04-25
topic: NestJS
draft: false
---

This processes controller entries with `@EventPattern({ action: 'XXXX' })`.
```typescript
import { Logger } from '@nestjs/common';
import { CustomTransportStrategy, MessageHandler, Server } from '@nestjs/microservices';
import { Context, SNSEvent, SNSEventRecord } from 'aws-lambda';

export class AWSSNSServer extends Server implements CustomTransportStrategy {
  event: SNSEvent;
  context: Context;

  constructor(event: SNSEvent, context: Context) {
    super();
    this.event = event;
    this.context = context;
  }

  /**
  * This method is triggered when you run "app.listen()".
  */
  async listen(callback: () => void) {
    for await (const record of this.getRecords()) {
      const messageAttributes = this.getMessageAttributes(record);
      const msg = this.getMessage(record);
      Logger.debug(messageAttributes, 'MessageAttributes');
      Logger.debug(msg, 'SNSMessage');
      Logger.log([...this.messageHandlers.keys()]);
      const handler = this.findMatchingHandler(messageAttributes);
      if (handler) {
        await handler(msg, this.context);
      } else {
        Logger.error(`No handler found.`);
      }
    }
    callback();
  }

  private findMatchingHandler(
  messageAttributes: Record<string, string>,
  ): MessageHandler<any, any, any> | undefined {
    for (const messageHandlerKeyInString of this.messageHandlers.keys()) {
      let eventPattern: Record<string, string>;
    
      try {
        eventPattern = JSON.parse(messageHandlerKeyInString);
      } catch (_e) {
        continue;
      }
    
      if (this.matchesEventPattern(eventPattern, messageAttributes)) {
        return this.messageHandlers.get(messageHandlerKeyInString);
      }
    }
    return undefined;
  }

  private matchesEventPattern(
  eventPattern: Record<string, string>,
  messageAttributes: Record<string, string>,
  ): boolean {
    for (const [key, value] of Object.entries(eventPattern)) {
      if (messageAttributes[key] !== value) {
        return false;
      }
    }
    return true;
  }

  /**
  * This method is triggered on application shutdown.
  */

  close() {
  return;
  }

  /**
  * This method converts all messageAttributes to an object of { [x:string]: string }
  * Example:
  * MessageAttributeMap: `[{ key2: { Type: String, Value: String }, key2: { Type: String, Value: String } }]`
  * Output: { key1:value1, key2:value2 }
  * @private
  * @param {SNSEventRecord} record
  * @return {*}  {string}
  * @memberof AWSSNSServer
  */

  private getMessageAttributes(record: SNSEventRecord): Record<string, string> {

    const messageAttributeMap = record.Sns.MessageAttributes;
    
    const returnObject: Record<string, string> = {};
    
    Object.keys(messageAttributeMap).forEach((key) => {
    
      returnObject[key] = messageAttributeMap[key].Value;
    
    });

    return returnObject;
  }

  /**
  * This returns the Message part of SQS
  */
  private getMessage(record: SNSEventRecord): string {
    return record.Sns.Message;
  }

  private getRecords(): SNSEventRecord[] {
    return this.event.Records;
  }
}
```