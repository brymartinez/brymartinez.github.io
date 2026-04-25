
---
title: AWS SSM Config Service on NestJS
description: Notes on integrating SSM secret fetching on ConfigService in NestJS.
pubDate: 2026-04-25
topic: NestJS
draft: false
---

```typescript
import { Injectable, Logger } from '@nestjs/common';
import {
  SSMClient,
  GetParameterCommand,
  GetParameterCommandInput,
} from '@aws-sdk/client-ssm';

@Injectable()
export class SSMConfigService {
  private SSM: SSMClient;
  private loaded = false;

  public async load(): Promise<void> {
    if (!this.loaded) {
      for (const [k, v] of Object.entries(process.env)) {
        if (typeof v === 'string' && v.substring(0, 4) === 'ssm:') {
          Logger.debug(`Converting ${v}`);
          const ssmParameterName: string = v.substring(4, v.length);
          const ssmResult: string = await this.loadValueFromSSM(
            ssmParameterName,
          );
          Logger.debug(`Converted ${v}`);
          process.env[k] = ssmResult;
        }
      }

      Logger.debug('SSM Configuration loaded.');
      this.loaded = true;
    }
  }

  private async loadValueFromSSM(name: string): Promise<string> {
    this.SSM = new SSMClient({});
    const input: GetParameterCommandInput = {
      Name: name,
      // WithDecryption: true,
    };
    const command: GetParameterCommand = new GetParameterCommand(input);
    const result = await this.SSM.send(command);
    return result.Parameter.Value;
  }
}
```