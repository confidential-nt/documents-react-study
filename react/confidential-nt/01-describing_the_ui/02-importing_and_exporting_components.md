## Importing and Exporting Components

### 들어가면서

한 컴포넌트는 커질 수 있음. 이때는 분리를 해야함. 이번 챕터에서는 분리를 해서 어떻게 export하고 import해오는지에 관해 배울 것임. (다만 아는 내용이 많으므로 유의사항에 관해서만 정리할 것.)

- 모듈을 내보내고 가져오는 법에 관한 설명은 [이 아티클](https://ko.javascript.info/import-export)에 잘 설명되어있음.

### export, import를 할 때 유의사항

1. export 방식에는 default export, named export 가 있음.
   - 전자는 어떤 파일에서 단 하나의 요소에만 사용 가능함. 후자는 여러 요소들에 사용 가능.
2. default export, named export를 한 파일에서 사용할 수 있지만 혼란을 방지하기 위해 한 파일에서는 하나의 방식만을 사용하는 등의 컨벤션을 정하기도 함.
3. default export는 해당 파일에 해당 요소가 하나만 있다는 것을 보장해주므로 import 해올 때 아무 이름이나 붙여도 됨. 단, 'import {...}' 이렇게 하면 안되고 'Import ... ' 이렇게 해야함.
4. named export는 import 해올 때, export를 앞에 붙인 요소의 이름과 같아야 함. 이게 바로 named export 라고 불리우는 이유.
