# BMP 이미지 그레이스케일 변환 소프트웨어

### base image
<img width="320" height="320" alt="brainct_001" src="https://github.com/user-attachments/assets/60efb670-acae-411a-8f13-886e434d0e8d" />

### grayscale
<img width="320" height="320" alt="output_grayscale" src="https://github.com/user-attachments/assets/1aae6d9f-ba9d-4989-940e-94e03c50c865" />

### Sobel_filter
<img width="320" height="320" alt="output_edge_sobel" src="https://github.com/user-attachments/assets/e85a4bd0-9e3a-4c0f-8efe-6aa3e77f3eba" />


### 문서 관리 정보
- 문서 ID: BMP-DO178C-001-Rev2
- 버전 & 날짜 : 2.0 / 2025년 9월 7일
- 분류: 설계 보증 레벨 D (DAL-D)
- 시스템: 항공 디스플레이 처리 장치 및 Verilog 메모리 인터페이스
- 주요 변경사항: MEM 파일 생성 기능 추가, 소스 코드 기반 세부사항 반영
------------------------------------------------------

## 1. 소프트웨어 인증 계획서 (PSAC)
#### 1.1 소프트웨어 개요
**소프트웨어 항목**: BMP 그레이스케일 변환기 + Verilog MEM 생성기
**기능**: 
- 630x630 24비트 컬러 BMP 이미지를 8비트 그레이스케일로 변환
- Verilog 시뮬레이션용 MEM 파일 생성 (16진수 형식)
- 항공 디스플레이 시스템 및 FPGA 개발 지원

**중요도 레벨**: DAL-D (경미한 고장 상황)

**입출력 파일**:
- 입력: `brainct_001.bmp` (24비트 컬러, 630x630)
- 출력1: `output_grayscale.bmp` (8비트 그레이스케일)
- 출력2: `output_image.mem` (Verilog 메모리 파일)

#### 1.2 소프트웨어 생명주기 프로세스
- 계획 수립 프로세스
- 소프트웨어 개발 프로세스
- 소프트웨어 검증 프로세스
- 소프트웨어 형상관리 프로세스
- 소프트웨어 품질보증 프로세스

#### 1.3 소프트웨어 생명주기 데이터
**제출 문서**:
- 소프트웨어 인증 계획서 (PSAC)
- 소프트웨어 요구사항 표준 (SRS)
- 소프트웨어 설계 표준 (SDS)
- 소프트웨어 코드 표준 (SCS)
- 소프트웨어 검증 계획 (SVP)
- 소프트웨어 형상관리 계획 (SCMP)

------------------------------------------------------
## 2. 소프트웨어 요구사항 표준 (SRS)
#### 2.1 상위레벨 요구사항

**HLR-001**: 이미지 형식 검증 및 크기 제한
- 소프트웨어는 입력 파일의 BMP 시그니처(0x4D42, "BM")를 검증해야 한다
- 소프트웨어는 24비트 BMP 형식임을 확인해야 한다 (bitCount = 24)
- 소프트웨어는 이미지 크기가 정확히 630x630 픽셀임을 검증해야 한다
- 소프트웨어는 압축되지 않은 BMP 파일만 처리해야 한다

**HLR-002**: 색공간 변환 및 정밀도
- 소프트웨어는 표준 휘도 공식을 사용하여 RGB 픽셀을 그레이스케일로 변환해야 한다: Y = 0.299×R + 0.587×G + 0.114×B
- 변환 결과는 uint8_t 형으로 캐스팅되어 0-255 범위를 유지해야 한다
- 픽셀의 공간적 위치와 순서를 변경 없이 유지해야 한다

**HLR-003**: 듀얼 출력 파일 생성
- 소프트웨어는 8비트 그레이스케일 BMP 파일을 생성해야 한다
- 소프트웨어는 256색 그레이스케일 팔레트를 포함해야 한다
- 소프트웨어는 Verilog 시뮬레이션용 MEM 파일을 추가로 생성해야 한다
- MEM 파일은 각 픽셀값을 2자리 16진수로 포맷하고 CRLF(0x0D 0x0A)로 구분해야 한다

**HLR-004**: 메모리 관리 및 안전성
- 소프트웨어는 동적 메모리 할당(malloc) 실패를 감지하고 처리해야 한다
- 소프트웨어는 할당된 모든 메모리를 프로그램 종료 전에 해제(free)해야 한다
- 이미지 데이터용 메모리와 그레이스케일 데이터용 메모리를 별도로 관리해야 한다

**HLR-005**: 포괄적 오류 처리
- 소프트웨어는 fopen_s() 함수의 반환값으로 파일 오류를 감지해야 한다
- 소프트웨어는 모든 파일 핸들을 적절히 닫아야 한다 (inFile, outFile, memOutFile)
- 오류 발생시 명확한 한국어 메시지를 제공해야 한다

**HLR-006**: BMP 구조 처리 (추가)
- 소프트웨어는 #pragma pack을 사용하여 구조체 패딩을 제거해야 한다
- 소프트웨어는 BMP의 행 패딩을 정확히 계산하고 처리해야 한다 ((4 - ((width * 3) % 4)) % 4)
- 소프트웨어는 BMP의 상하반전 구조를 고려하여 데이터를 처리해야 한다

#### 2.2 저수준 요구사항

**LLR-001**: BMP 헤더 상세 검증
- BMPFileHeader.type이 0x4D42인지 확인
- BMPInfoHeader.bitCount가 24인지 검증
- BMPInfoHeader.width가 630인지 확인
- BMPInfoHeader.height가 630인지 확인
- 헤더 정보를 콘솔에 출력하여 디버깅 지원

**LLR-002**: 픽셀 데이터 정확한 읽기
- fileHeader.offset 위치로 파일 포인터 이동 (fseek)
- BMP의 하단-상단 순서로 픽셀 데이터 읽기 (y = HEIGHT-1 부터 0까지)
- 각 행의 패딩 바이트를 SEEK_CUR로 건너뛰기
- RGB 구조체 순서 (blue, green, red) 준수

**LLR-003**: 그레이스케일 변환 함수
- rgbToGrayscale() 함수 구현
- 공식: (uint8_t)(0.299 * pixel.red + 0.587 * pixel.green + 0.114 * pixel.blue)
- 부동소수점 연산 후 정수 캐스팅
- 입력: RGB 구조체, 출력: uint8_t

**LLR-004**: 8비트 BMP 출력 생성
- newPadding 계산: (4 - (width % 4)) % 4
- 256색 팔레트 생성 (각 색상 i에 대해 BGRA: i,i,i,0)
- 그레이스케일 데이터를 하단부터 상단 순으로 출력
- 적절한 패딩 바이트 추가

**LLR-005**: MEM 파일 생성 (신규)
- 텍스트 모드("w")로 MEM 파일 생성
- fprintf()로 각 픽셀값을 "%02X\r\n" 형식으로 출력
- 396,900개 라인 생성 (630×630 픽셀)
- Windows 형식 줄바꿈 사용 (CRLF)

**LLR-006**: 메모리 할당 순서 및 해제
- RGB* imageData 할당: IMAGE_WIDTH × IMAGE_HEIGHT × sizeof(RGB)
- uint8_t* grayscaleData 할당: IMAGE_WIDTH × IMAGE_HEIGHT
- 할당 실패시 이전 할당된 메모리 해제
- 프로그램 종료 전 free(imageData), free(grayscaleData) 실행

------------------------------------------------

## 3. 소프트웨어 설계 표준 (SDS)
#### 3.1 아키텍처 설계

**개선된 모듈 구조**:
```
BMP_Processor_v2
├── File_Manager
│   ├── Input_Validator
│   ├── BMP_Reader
│   └── Dual_Output_Generator
│       ├── BMP_Writer
│       └── MEM_Writer
├── Image_Processor
│   ├── Color_Converter
│   └── Memory_Manager
├── Header_Analyzer
└── Error_Handler
```

#### 3.2 데이터 구조 설계

**구조체 정의 (pragma pack 적용)**:
```c
#pragma pack(push, 1)
typedef struct {
    uint16_t type;        // 0x4D42 ("BM")
    uint32_t size;        // 전체 파일 크기
    uint16_t reserved1;   // 0
    uint16_t reserved2;   // 0  
    uint32_t offset;      // 픽셀 데이터 시작 위치
} BMPFileHeader;

typedef struct {
    uint32_t size;        // 40 (헤더 크기)
    int32_t width;        // 630
    int32_t height;       // 630
    uint16_t planes;      // 1
    uint16_t bitCount;    // 24 (입력) / 8 (출력)
    uint32_t compression; // 0 (무압축)
    uint32_t sizeImage;   // 이미지 데이터 크기
    int32_t xPelsPerMeter;// 해상도 (미사용)
    int32_t yPelsPerMeter;// 해상도 (미사용)
    uint32_t clrUsed;     // 사용된 색상 수
    uint32_t clrImportant;// 중요한 색상 수
} BMPInfoHeader;

typedef struct {
    uint8_t blue;         // B 값 (0-255)
    uint8_t green;        // G 값 (0-255)
    uint8_t red;          // R 값 (0-255)
} RGB;
#pragma pack(pop)
```

#### 3.3 주요 함수 설계

**핵심 함수**:
- `main()`: 전체 프로세스 제어 및 오류 처리
- `rgbToGrayscale(RGB pixel)`: RGB→그레이스케일 변환
- 파일 I/O: `fopen_s()`, `fread()`, `fwrite()`, `fprintf()`
- 메모리 관리: `malloc()`, `free()`
- 파일 탐색: `fseek()`

**변환 공식 함수**:
```c
uint8_t rgbToGrayscale(RGB pixel) {
    return (uint8_t)(0.299 * pixel.red + 0.587 * pixel.green + 0.114 * pixel.blue);
}
```

#### 3.4 인터페이스 설계

**파일 인터페이스**:
- 입력: `brainct_001.bmp` (하드코딩)
  - 형식: 24비트 컬러 BMP
  - 크기: 630×630 픽셀
  - 압축: 없음

- 출력1: `output_grayscale.bmp`
  - 형식: 8비트 그레이스케일 BMP
  - 팔레트: 256색 (0-255, 각각 BGRA 형식)
  - 패딩: 2바이트 (630 % 4 = 2)

- 출력2: `output_image.mem`
  - 형식: ASCII 텍스트
  - 내용: 16진수 값 (00-FF)
  - 구분자: CRLF (Windows 형식)
  - 총 라인 수: 396,900개

**메모리 사용량**:
- imageData: 630 × 630 × 3 = 1,191,780 바이트
- grayscaleData: 630 × 630 × 1 = 396,900 바이트
- 총 메모리: 약 1.59 MB

--------------------------------------------

## 4. 소프트웨어 코드 표준 (SCS)
#### 4.1 코딩 규칙 (실제 구현 기준)

**명명 규칙**:
- 상수: UPPER_CASE (`IMAGE_WIDTH`, `IMAGE_HEIGHT`)
- 변수: camelCase (`imageData`, `grayscaleData`, `memOutFile`)
- 구조체: PascalCase (`BMPFileHeader`, `BMPInfoHeader`, `RGB`)
- 함수: camelCase (`rgbToGrayscale`)
- 파일명: snake_case (`brainct_001.bmp`, `output_grayscale.bmp`)

**실제 사용된 변수명**:
```c
const int IMAGE_WIDTH = 630;
const int IMAGE_HEIGHT = 630;
RGB* imageData;
uint8_t* grayscaleData;  
FILE* inFile, *outFile, *memOutFile;
int padding, newPadding;
int rowSize, newRowSize;
int paletteSize = 256 * 4;
```

#### 4.2 메모리 안전성 규칙

**동적 메모리 할당 패턴**:
```c
// 할당
RGB* imageData = (RGB*)malloc(IMAGE_WIDTH * IMAGE_HEIGHT * sizeof(RGB));
if (imageData == NULL) {
    // 오류 처리 및 기존 리소스 해제
    return 1;
}

// 사용 후 해제
free(imageData);
free(grayscaleData);
```

**파일 핸들 관리**:
```c
// 안전한 파일 열기
if (fopen_s(&inFile, inputFile, "rb") != 0 || inFile == NULL) {
    printf("입력 파일을 열 수 없습니다: %s\n", inputFile);
    return 1;
}

// 사용 후 닫기
fclose(inFile);
fclose(outFile); 
fclose(memOutFile);
```

#### 4.3 오류 처리 패턴

**체계적 오류 검증**:
1. 파일 존재 여부 (`fopen_s` 반환값)
2. BMP 시그니처 검증 (`fileHeader.type != 0x4D42`)
3. 비트 깊이 확인 (`infoHeader.bitCount != 24`)
4. 이미지 크기 검증 (`width != 630 || height != 630`)
5. 메모리 할당 실패 (`malloc` 반환값)

## 5. 소프트웨어 검증 계획 (SVP)
#### 5.1 검증 목표
- 모든 요구사항의 올바른 구현 확인
- BMP 및 MEM 파일 형식 준수 검증
- 메모리 안전성 및 리소스 관리 검증
- 오류 조건에서의 적절한 동작 확인

#### 5.2 검증 방법

**정적 분석**:
- 코드 리뷰 (pragma pack 사용, 포인터 안전성)
- MISRA C 규칙 준수 검사
- 메모리 누수 정적 검사

**동적 테스트**:
- 단위 테스트 (`rgbToGrayscale` 함수)
- 통합 테스트 (전체 변환 프로세스)
- 파일 형식 검증 테스트

#### 5.3 상세 테스트 케이스

**TC-001**: 정상 처리 경로
- **입력**: 유효한 630×630 24비트 BMP 파일
- **예상 결과**: 
  - `output_grayscale.bmp` 생성 (8비트, 256색 팔레트)
  - `output_image.mem` 생성 (396,900줄, 16진수 형식)
  - 메모리 정상 해제
- **검증 방법**: 출력 파일 크기 및 헤더 구조 확인

**TC-002**: 잘못된 BMP 시그니처
- **입력**: type 필드가 0x4D42가 아닌 파일
- **예상 결과**: "유효하지 않은 BMP 파일입니다." 메시지 출력
- **검증**: 프로그램 종료 코드 1

**TC-003**: 잘못된 비트 깊이
- **입력**: 16비트 또는 32비트 BMP 파일
- **예상 결과**: "24비트 BMP 파일이 아닙니다. (현재: X비트)" 메시지
- **검증**: 적절한 오류 메시지 형식

**TC-004**: 잘못된 이미지 크기
- **입력**: 512×512 또는 1024×768 BMP 파일
- **예상 결과**: "이미지 크기가 630x630가 아닙니다. (현재: WxH)" 메시지
- **검증**: 크기 정보 정확성

**TC-005**: 메모리 할당 실패 시뮬레이션
- **테스트 방법**: 메모리 제한 환경에서 실행
- **예상 결과**: "메모리 할당 실패" 메시지 및 적절한 정리
- **검증**: 메모리 누수 없음

**TC-006**: 파일 I/O 오류
- **테스트**: 읽기 전용 디렉토리에 출력 파일 생성 시도
- **예상 결과**: 적절한 I/O 오류 메시지
- **검증**: 파일 핸들 누수 없음

**TC-007**: MEM 파일 형식 검증 (신규)
- **검증 항목**:
  - 정확히 396,900줄 생성
  - 각 줄이 2자리 16진수 + CRLF 형식
  - 값 범위 00-FF 준수
- **방법**: 외부 Verilog 시뮬레이터로 로드 테스트

**TC-008**: 그레이스케일 변환 정확성
- **테스트 케이스**: 
  - 순수 빨강 (255,0,0) → 76 (0x4C)
  - 순수 녹색 (0,255,0) → 150 (0x96)  
  - 순수 파랑 (0,0,255) → 29 (0x1D)
  - 흰색 (255,255,255) → 255 (0xFF)
  - 검정 (0,0,0) → 0 (0x00)

#### 5.4 커버리지 기준
- **명령문 커버리지**: 100%
- **분기 커버리지**: 100% (모든 if-else, 오류 조건)
- **함수 커버리지**: 100%
- **MC/DC**: 해당 없음 (DAL-D 등급)

-----------------------------------------------

## 6. 소프트웨어 형상관리 계획 (SCMP)
#### 6.1 형상관리 대상 (업데이트)

**소스 코드**:
- `bmp_converter.c` (주 소스 파일)
- 컴파일 스크립트 및 Makefile

**입출력 파일**:
- `brainct_001.bmp` (테스트 입력)
- `output_grayscale.bmp` (BMP 출력)
- `output_image.mem` (Verilog MEM 출력)

**문서화**:
- DO-178C 인증 문서 세트
- 사용자 가이드
- 테스트 결과 리포트

#### 6.2 버전 관리 규칙

**버전 번호 체계**: MAJOR.MINOR.PATCH
- MAJOR: 주요 기능 변경 (MEM 파일 생성 추가 = v2.0)
- MINOR: 기능 개선 및 추가
- PATCH: 버그 수정

**현재 버전**: v2.0.0
- v1.0.0: 기본 BMP 그레이스케일 변환
- v2.0.0: MEM 파일 생성 기능 추가

-------------------------------------------------------------

## 7. 추적성 매트릭스 (업데이트)

| 요구사항 ID | 설계 요소 | 코드 구현 | 테스트 케이스 | 비고 |
|:-----:|:-----:|:-----:|:-----:|:-----:|
| HLR-001 | File_Validator | `fread()`, 헤더 검증 로직 | TC-001, TC-002, TC-003, TC-004 | BMP 검증 |
| HLR-002 | Color_Converter | `rgbToGrayscale()` | TC-001, TC-008 | 변환 공식 |
| HLR-003 | Dual_Output_Generator | BMP 생성 + MEM 생성 루틴 | TC-001, TC-007 | **신규**: MEM 파일 |
| HLR-004 | Memory_Manager | `malloc()`, `free()` | TC-005 | 동적 메모리 |
| HLR-005 | Error_Handler | 모든 오류 처리 루틴 | TC-002~TC-006 | 포괄적 오류 처리 |
| HLR-006 | Header_Analyzer | pragma pack, 패딩 계산 | TC-001 | **신규**: 구조체 정렬 |
| LLR-001 | BMP_Reader | 헤더 읽기 + 검증 | TC-002, TC-003, TC-004 | 상세 검증 |
| LLR-002 | BMP_Reader | 픽셀 데이터 읽기 루틴 | TC-001 | 하단-상단 순서 |
| LLR-003 | Color_Converter | 공식 구현 | TC-008 | 부동소수점 정밀도 |
| LLR-004 | BMP_Writer | 8비트 BMP 생성 | TC-001 | 팔레트 포함 |
| LLR-005 | MEM_Writer | `fprintf("%02X\r\n")` | TC-007 | **신규**: Verilog 형식 |
| LLR-006 | Memory_Manager | 순차적 할당/해제 | TC-005 | 메모리 안전성 |

---------------------------------------------------------

## 8. 성능 및 리소스 분석 (신규)

#### 8.1 메모리 사용량
- **컴파일 타임**: 구조체 정의 (pragma pack 적용)
- **런타임 최대**: 1,588,680 바이트 (약 1.59 MB)
  - imageData: 1,191,780 바이트
  - grayscaleData: 396,900 바이트
- **메모리 해제**: 프로그램 종료 전 완전 해제

#### 8.2 파일 I/O 성능
- **읽기 작업**: 1회 (BMP 파일)
- **쓰기 작업**: 2회 (BMP + MEM)
- **예상 처리 시간**: 100ms 미만 (일반적인 SSD 환경)

#### 8.3 디스크 사용량
- **입력**: ~1.2 MB (24비트 BMP)
- **출력1**: ~398 KB (8비트 BMP + 팔레트)
- **출력2**: ~1.2 MB (MEM 텍스트 파일)
- **총 요구량**: ~2.8 MB

## 9. 인증 결론

본 BMP 그레이스케일 변환 소프트웨어 v2.0은 DO-178C DAL-D 수준의 모든 요구사항을 충족합니다. 

**주요 개선사항**:
1. **이중 출력 지원**: BMP와 MEM 파일 동시 생성
2. **향상된 오류 처리**: 모든 파일 I/O 및 메모리 오류 상황 대응
3. **정밀한 구조체 관리**: pragma pack을 통한 바이트 정렬 보장
4. **Verilog 호환성**: FPGA 개발 워크플로우 지원

**검증 완료 항목**:
- ✅ 모든 HLR/LLR 요구사항 구현
- ✅ 메모리 안전성 확보
- ✅ 포괄적 오류 처리
- ✅ 다중 출력 형식 지원
- ✅ 성능 요구사항 충족

**인증 승인**:
- 개발팀장: [서명 대기]
- 품질보증 관리자: [서명 대기]  
- 인증 담당자: [서명 대기]

**최종 승인 날짜**: 2025년 9월 7일

---

*본 문서는 실제 C 소스 코드를 기반으로 작성되었으며, 모든 구현 세부사항이 요구사항과 일치함을 확인하였습니다.*




