# main.py
# 블로그 키워드 추출 및 제목 생성 메인 스크립트

import sys
import os
from datetime import datetime

# 모듈 임포트
from keyword_extractor import extract_keywords, get_keyword_list
from title_generator import generate_titles_with_info
from excel_exporter import export_to_excel, create_summary_sheet

def print_header():
    """프로그램 헤더 출력"""
    print("\n" + "=" * 60)
    print("   📝 블로그 키워드 추출 & 제목 생성기")
    print("   Blog Keyword Extractor & Title Generator")
    print("=" * 60)

def print_menu():
    """메뉴 출력"""
    print("\n[메뉴]")
    print("  1. 리뷰 분석 (키워드 추출 + 제목 생성 + 엑셀 저장)")
    print("  2. 키워드만 추출")
    print("  3. 제목만 생성 (키워드 직접 입력)")
    print("  4. 저장된 파일 확인")
    print("  5. 요약 리포트 생성")
    print("  0. 종료")
    print("-" * 40)

def get_multiline_input(prompt):
    """여러 줄 입력 받기"""
    print(prompt)
    print("(입력 완료 후 빈 줄에서 Enter를 두 번 누르세요)")
    print("-" * 40)
    
    lines = []
    empty_count = 0
    
    while True:
        try:
            line = input()
            if line == "":
                empty_count += 1
                if empty_count >= 2:
                    break
                lines.append(line)
            else:
                empty_count = 0
                lines.append(line)
        except EOFError:
            break
    
    return "\n".join(lines).strip()

def analyze_review(review_text, num_keywords=10, num_titles=5, save_to_excel=True):
    """
    리뷰 분석 전체 프로세스 실행
    
    Args:
        review_text (str): 분석할 리뷰 텍스트
        num_keywords (int): 추출할 키워드 수
        num_titles (int): 생성할 제목 수
        save_to_excel (bool): 엑셀 저장 여부
    
    Returns:
        dict: 분석 결과
    """
    print("\n🔍 분석 중...")
    
    # 1. 키워드 추출
    print("  [1/3] 키워드 추출 중...")
    keywords_with_count = extract_keywords(review_text, top_n=num_keywords)
    keyword_list = [kw for kw, count in keywords_with_count]
    
    if not keyword_list:
        print("  ⚠️  키워드를 추출할 수 없습니다. 리뷰 내용을 확인해주세요.")
        return None
    
    # 2. 제목 생성
    print("  [2/3] 제목 생성 중...")
    title_result = generate_titles_with_info(keyword_list, count=num_titles)
    
    # 3. 엑셀 저장
    filepath = None
    if save_to_excel:
        print("  [3/3] 엑셀 저장 중...")
        filepath = export_to_excel(
            keywords=keywords_with_count,
            titles=title_result['titles'],
            category=title_result['category'],
            review_text=review_text
        )
    
    result = {
        'keywords': keywords_with_count,
        'category': title_result['category'],
        'titles': title_result['titles'],
        'filepath': filepath
    }
    
    return result

def display_results(result):
    """분석 결과 출력"""
    print("\n" + "=" * 60)
    print("📊 분석 결과")
    print("=" * 60)
    
    # 카테고리
    category_names = {
        'general': '일반',
        'restaurant': '맛집/음식',
        'medical': '의료/병원',
        'product': '제품/상품',
        'place': '장소/여행'
    }
    print(f"\n📁 감지된 카테고리: {category_names.get(result['category'], result['category'])}")
    
    # 키워드
    print("\n🔑 추출된 키워드:")
    for i, (keyword, count) in enumerate(result['keywords'], 1):
        print(f"   {i}. {keyword} ({count}회)")
    
    # 제목
    print("\n📝 생성된 블로그 제목:")
    for i, title in enumerate(result['titles'], 1):
        print(f"   {i}. {title}")
    
    # 저장 경로
    if result['filepath']:
        print(f"\n💾 저장 완료: {result['filepath']}")
    
    print("\n" + "=" * 60)

def menu_analyze_review():
    """메뉴 1: 리뷰 전체 분석"""
    review_text = get_multiline_input("\n리뷰 내용을 입력하세요:")
    
    if not review_text:
        print("⚠️  리뷰 내용이 비어있습니다.")
        return
    
    # 옵션 입력
    try:
        num_keywords = input("\n추출할 키워드 수 (기본 10): ").strip()
        num_keywords = int(num_keywords) if num_keywords else 10
        
        num_titles = input("생성할 제목 수 (기본 5): ").strip()
        num_titles = int(num_titles) if num_titles else 5
    except ValueError:
        print("⚠️  숫자를 입력해주세요. 기본값으로 진행합니다.")
        num_keywords = 10
        num_titles = 5
    
    # 분석 실행
    result = analyze_review(review_text, num_keywords, num_titles)
    
    if result:
        display_results(result)

def menu_extract_keywords_only():
    """메뉴 2: 키워드만 추출"""
    review_text = get_multiline_input("\n리뷰 내용을 입력하세요:")
    
    if not review_text:
        print("⚠️  리뷰 내용이 비어있습니다.")
        return
    
    try:
        num_keywords = input("\n추출할 키워드 수 (기본 10): ").strip()
        num_keywords = int(num_keywords) if num_keywords else 10
    except ValueError:
        num_keywords = 10
    
    keywords = extract_keywords(review_text, top_n=num_keywords)
    
    print("\n" + "=" * 40)
    print("🔑 추출된 키워드:")
    print("=" * 40)
    for i, (keyword, count) in enumerate(keywords, 1):
        print(f"  {i}. {keyword} ({count}회)")

def menu_generate_titles_only():
    """메뉴 3: 제목만 생성"""
    print("\n키워드를 입력하세요 (쉼표로 구분):")
    keywords_input = input("> ").strip()
    
    if not keywords_input:
        print("⚠️  키워드가 비어있습니다.")
        return
    
    keywords = [kw.strip() for kw in keywords_input.split(",") if kw.strip()]
    
    try:
        num_titles = input("\n생성할 제목 수 (기본 5): ").strip()
        num_titles = int(num_titles) if num_titles else 5
    except ValueError:
        num_titles = 5
    
    result = generate_titles_with_info(keywords, count=num_titles)
    
    print("\n" + "=" * 40)
    print(f"📁 감지된 카테고리: {result['category']}")
    print("\n📝 생성된 블로그 제목:")
    print("=" * 40)
    for i, title in enumerate(result['titles'], 1):
        print(f"  {i}. {title}")

def menu_check_files():
    """메뉴 4: 저장된 파일 확인"""
    output_dir = "output"
    
    if not os.path.exists(output_dir):
        print("\n⚠️  저장된 파일이 없습니다.")
        return
    
    files = [f for f in os.listdir(output_dir) if f.endswith('.xlsx')]
    
    if not files:
        print("\n⚠️  저장된 엑셀 파일이 없습니다.")
        return
    
    print("\n" + "=" * 40)
    print("💾 저장된 파일 목록:")
    print("=" * 40)
    for i, filename in enumerate(files, 1):
        filepath = os.path.join(output_dir, filename)
        file_size = os.path.getsize(filepath)
        mod_time = datetime.fromtimestamp(os.path.getmtime(filepath))
        print(f"  {i}. {filename}")
        print(f"     크기: {file_size:,} bytes | 수정일: {mod_time.strftime('%Y-%m-%d %H:%M')}")

def menu_create_summary():
    """메뉴 5: 요약 리포트 생성"""
    output_dir = "output"
    
    if not os.path.exists(output_dir):
        print("\n⚠️  저장된 파일이 없습니다.")
        return
    
    files = [f for f in os.listdir(output_dir) if f.endswith('.xlsx')]
    
    if not files:
        print("\n⚠️  저장된 엑셀 파일이 없습니다.")
        return
    
    print("\n파일 선택:")
    for i, f in enumerate(files, 1):
        print(f"  {i}. {f}")
    
    try:
        choice = int(input("\n번호 입력: ")) - 1
        if 0 <= choice < len(files):
            filepath = create_summary_sheet(files[choice])
            print(f"\n✅ 요약 시트가 추가되었습니다: {filepath}")
        else:
            print("⚠️  잘못된 선택입니다.")
    except ValueError:
        print("⚠️  숫자를 입력해주세요.")

def run_interactive():
    """대화형 모드 실행"""
    print_header()
    
    while True:
        print_menu()
        
        try:
            choice = input("선택 (0-5): ").strip()
            
            if choice == "0":
                print("\n👋 프로그램을 종료합니다. 감사합니다!")
                break
            elif choice == "1":
                menu_analyze_review()
            elif choice == "2":
                menu_extract_keywords_only()
            elif choice == "3":
                menu_generate_titles_only()
            elif choice == "4":
                menu_check_files()
            elif choice == "5":
                menu_create_summary()
            else:
                print("⚠️  잘못된 선택입니다. 0-5 사이의 숫자를 입력해주세요.")
        
        except KeyboardInterrupt:
            print("\n\n👋 프로그램을 종료합니다.")
            break
        except Exception as e:
            print(f"\n❌ 오류 발생: {e}")

def run_single_analysis(review_text):
    """단일 분석 실행 (비대화형)"""
    result = analyze_review(review_text)
    if result:
        display_results(result)
    return result

if __name__ == "__main__":
    # 커맨드라인 인자 확인
    if len(sys.argv) > 1:
        # 파일에서 리뷰 읽기
        if sys.argv[1] == "--file" and len(sys.argv) > 2:
            filepath = sys.argv[2]
            if os.path.exists(filepath):
                with open(filepath, 'r', encoding='utf-8') as f:
                    review_text = f.read()
                run_single_analysis(review_text)
            else:
                print(f"⚠️  파일을 찾을 수 없습니다: {filepath}")
        # 직접 텍스트 입력
        elif sys.argv[1] == "--text":
            review_text = " ".join(sys.argv[2:])
            run_single_analysis(review_text)
        # 도움말
        elif sys.argv[1] in ["--help", "-h"]:
            print("사용법:")
            print("  python main.py              # 대화형 모드")
            print("  python main.py --file FILE  # 파일에서 리뷰 읽기")
            print("  python main.py --text TEXT  # 직접 텍스트 분석")
        else:
            print("알 수 없는 옵션입니다. --help를 참조하세요.")
    else:
        # 대화형 모드 실행
        run_interactive()
