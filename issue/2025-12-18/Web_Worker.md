# web worker

회사 프로젝트에서 pptxgenjs로 ppt를 다운로드하는 기능을 개발했다. 문제는 이미지 파일의 용량과 갯수가 너무 많아 다운로드에 시간이 오래 걸린다는 점이었다. 처음에 개발할 때 백에서 처리하는 방식이었다면 좋았을텐데 오래전부터 프론트에서 처리하기도 했고, 시간 부족 이슈로, 이를 해결하기 위한 방안으로  base64로 변환하는 로직에 Web worker를 도입하고, 이미지를 배치 처리하는 방식을 적용했다. 

- 웹 워커: 브라우저의 메인 스레드와 별개로 작동되는 스레드를 생성할 수 있다. 웹 워커로 생성한 스레드는 브라우저 렌더링 같은 메인 스레드의 작업을 방해하지 않고, 새로운 스레드에서 스크립트를 실행하는 간단한 방법을 제공한다. 무거운 작업을 분리된 스레드에서 처리하면 주 스레드(보통 UI 스레드)가 멈추거나 느려지지 않고 동작할 수 있다.
- 사용방법: 백그라운드 스레드에서 돌리고자 하는 코드를 개별 파일에 두고, 메인 스레드에서는 해당 파일을 Worker로 선언해서 데이터와 연산 결과를 주고 받는다. 워커 스레드와 메인스레드는 메시지 시스템으로 데이터를 교환한다. postMessage를 통해 전송하고, onmessage 이벤트 처리기를 통해 수신한다.
- 예외: DOM 직접 조작 불가, window 일부 메서드와 속성 사용 불가

다음과 같이 별도의 파일을 만들어 사용했다(코드의 일부)

I developed a feature in a company project that allows users to download PowerPoint files using **pptxgenjs**.

The main issue was that the download took a long time because there were too many image files and their sizes were large. Ideally, this should have been handled on the backend from the beginning, but the logic had long been implemented on the frontend. Due to time constraints, we needed a frontend-based solution.

To address this, I integrated Web Workers into the Base64 conversion logic and applied **batch processing** for images. This significantly improved performance by preventing heavy processing from blocking the main thread.

- **Web Workers**: Web Workers allow you to create threads that run separately from the browser’s main thread. Scripts executed in a worker do not interfere with rendering or other main-thread tasks. By offloading heavy computations to a worker, the main (UI) thread remains responsive and does not freeze or slow down.
- **How it works**: The code that needs to run in the background is placed in a separate file. The main thread creates a Worker from that file and exchanges data with it. Communication between the main thread and the worker happens through a message-based system, using `postMessage` to send data and the `onmessage` event handler to receive results.
- **Limitations**: Web Workers cannot directly manipulate the DOM, and some `window` methods and properties are not available.

I implemented this by creating a separate worker file, as shown below (partial code).

```tsx
// workers/imageLoader.worker.ts

self.onmessage = async (e) => {
  const { url, index } = e.data;

  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    const blob = await response.blob();

    const base64 = await new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.onloadend = () => resolve(reader.result);
      reader.onerror = () => reject(new Error('FileReader failed'));
      reader.readAsDataURL(blob);
    });

    self.postMessage({
      type: 'COMPLETE',
      index,
      url,
      base64
    });

  } catch (error) {
    self.postMessage({
      type: 'ERROR',
      index,
      url,
      error: error.message || 'Unknown error'
    });
  }
};
```

```tsx
class ImageWorkerPool {
  workers: Worker[] = [];
  maxWorkers: number = navigator.hardwareConcurrency || 4;

  getWorker(): Worker {
    if (this.workers.length < this.maxWorkers) {
      const worker = new Worker(
        new URL('../workers/imageLoader.worker.ts?worker', import.meta.url)
      );

      this.workers.push(worker);
      return worker;
    }
    return this.workers[Math.floor(Math.random() * this.workers.length)];
  }

  terminate(): void {
    this.workers.forEach(w => w.terminate());
    this.workers = [];
  }
}
```