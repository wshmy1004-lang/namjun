<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>회비납부현황 관리 시스템</title>
    <style>
        body { font-family: 'Malgun Gothic', sans-serif; background-color: #f4f7f6; display: flex; flex-direction: column; align-items: center; padding: 20px; }
        .container { width: 450px; background: white; padding: 25px; border: 1px solid #333; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
        h2 { text-align: center; text-decoration: underline; margin-bottom: 20px; }
       
        table { width: 100%; border-collapse: collapse; margin-bottom: 10px; }
        th, td { border: 1px solid #333; height: 35px; padding: 5px 10px; }
        th { background-color: #f8f9fa; width: 35%; text-align: left; }
       
        input { width: 100%; border: 1px solid #ddd; padding: 5px; font-size: 15px; box-sizing: border-box; background-color: #fff; }
        input:focus { background-color: #fff3cd; border-color: #ffa500; outline: none; }
       
        .year-header { background-color: #e9ecef; text-align: center; font-weight: bold; }
        .total-row { background-color: #fff3cd; font-weight: bold; text-align: right; }

        .button-group { display: flex; border: 1px solid #333; margin-top: 15px; }
        .btn { flex: 1; padding: 10px; border: none; background: white; cursor: pointer; font-size: 14px; border-right: 1px solid #333; }
        .btn:last-child { border-right: none; }
        .btn:hover { background-color: #f0f0f0; }

        /* 하단 리스트 영역 및 너비 조절 */
        #data-list-section { margin-top: 30px; width: 900px; background: white; padding: 20px; border: 1px solid #333; }
        .search-box { width: 100%; padding: 10px; margin-bottom: 10px; border: 1px solid #ccc; box-sizing: border-box; }
       
        .list-table { width: 100%; border-collapse: collapse; font-size: 14px; table-layout: fixed; }
        .list-table th, .list-table td { text-align: center; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; border: 1px solid #ccc; }
        .list-table th { background: #eee; height: 40px; }
        .list-table td { height: 35px; }

        /* 칸 너비 상세 조절 */
        .col-name { width: 70px; }  
        .col-pos  { width: 70px; }  
        .col-loc  { width: 80px; }  
        .col-amt  { width: 120px; }  
        .col-total { width: 130px; }
        .col-action { width: 120px; } /* 수정/삭제 버튼 칸 */

        .edit-btn { background: #ffc107; border: none; padding: 4px 8px; cursor: pointer; border-radius: 3px; margin-right: 2px; }
        .del-btn { background: #dc3545; color: white; border: none; padding: 4px 8px; cursor: pointer; border-radius: 3px; }

        @media print { .button-group, #data-list-section, .search-box { display: none; } }
    </style>
</head>
<body>

<div class="container">
    <h2 id="form-title">회비납부현황</h2>
    <input type="hidden" id="edit-index" value="-1"> <table>
        <tr><th>직분</th><td><input type="text" id="position"></td></tr>
        <tr><th>지방</th><td><input type="text" id="loc1"></td></tr>
        <tr><th>교회</th><td><input type="text" id="loc2"></td></tr>
        <tr><th>연회비</th><td><input type="text" id="loc3"></td></tr>
        <tr><th>이름</th><td><input type="text" id="name"></td></tr>
       
        <tr><td colspan="2" class="year-header">2025년도</td></tr>
        <tr><th>납부일자</th><td><input type="date" id="date2025"></td></tr>
        <tr><th>납부액</th><td><input type="number" id="amt2025" oninput="calcTotal()" placeholder="0"></td></tr>
       
        <tr><td colspan="2" class="year-header">2026년도</td></tr>
        <tr><th>납부일자</th><td><input type="date" id="date2026"></td></tr>
        <tr><th>납부액</th><td><input type="number" id="amt2026" oninput="calcTotal()" placeholder="0"></td></tr>
       
        <tr class="total-row">
            <th>합계액</th>
            <td id="total-amount" style="text-align: right; padding-right: 15px;">0원</td>
        </tr>
    </table>

    <div class="button-group">
        <button class="btn" id="save-btn" onclick="saveData()">저장하기</button>
        <button class="btn" onclick="window.print()">인쇄</button>
        <button class="btn" onclick="resetForm()">새로고침</button>
    </div>
</div>

<div id="data-list-section">
    <h3>데이터리스트 (검색)</h3>
    <input type="text" class="search-box" id="search" onkeyup="filterList()" placeholder="이름으로 검색하세요...">
    <table class="list-table">
        <thead>
            <tr>
                <th class="col-name">이름</th>
                <th class="col-pos">직분</th>
                <th class="col-loc">지방</th>
                <th class="col-amt">2025 납부</th>
                <th class="col-amt">2026 납부</th>
                <th class="col-total">총액</th>
                <th class="col-action">관리</th>
            </tr>
        </thead>
        <tbody id="listBody"></tbody>
    </table>
</div>

<script>
    function calcTotal() {
        const amt25 = parseInt(document.getElementById('amt2025').value) || 0;
        const amt26 = parseInt(document.getElementById('amt2026').value) || 0;
        document.getElementById('total-amount').innerText = (amt25 + amt26).toLocaleString() + "원";
    }

    // 데이터 저장 (추가 및 수정 공용)
    function saveData() {
        const nameVal = document.getElementById('name').value;
        const posVal = document.getElementById('position').value;
        const loc1Val = document.getElementById('loc1').value;
        const loc2Val = document.getElementById('loc2').value;
        const loc3Val = document.getElementById('loc3').value;
        const date25 = document.getElementById('date2025').value;
        const a25 = document.getElementById('amt2025').value || 0;
        const date26 = document.getElementById('date2026').value;
        const a26 = document.getElementById('amt2026').value || 0;
        const editIdx = document.getElementById('edit-index').value;

        if(!nameVal) { alert("이름을 입력해주세요."); return; }

        const member = {
            name: nameVal, position: posVal,
            loc1: loc1Val, loc2: loc2Val, loc3: loc3Val,
            date2025: date25, amt2025: a25,
            date2026: date26, amt2026: a26
        };

        let list = JSON.parse(localStorage.getItem('memberList')) || [];

        if (editIdx === "-1") {
            list.push(member);
            alert("저장되었습니다.");
        } else {
            list[editIdx] = member;
            alert("수정되었습니다.");
        }
       
        localStorage.setItem('memberList', JSON.stringify(list));
        loadList();
        resetForm();
    }

    // 데이터 삭제 기능
    function deleteData(index) {
        if(confirm("정말 삭제하시겠습니까?")) {
            let list = JSON.parse(localStorage.getItem('memberList')) || [];
            list.splice(index, 1);
            localStorage.setItem('memberList', JSON.stringify(list));
            loadList();
        }
    }

    // 데이터 수정 모드로 전환
    function editData(index) {
        let list = JSON.parse(localStorage.getItem('memberList')) || [];
        const m = list[index];

        document.getElementById('name').value = m.name;
        document.getElementById('position').value = m.position;
        document.getElementById('loc1').value = m.loc1;
        document.getElementById('loc2').value = m.loc2;
        document.getElementById('loc3').value = m.loc3;
        document.getElementById('date2025').value = m.date2025;
        document.getElementById('amt2025').value = m.amt2025;
        document.getElementById('date2026').value = m.date2026;
        document.getElementById('amt2026').value = m.amt2026;
       
        document.getElementById('edit-index').value = index;
        document.getElementById('save-btn').innerText = "수정완료";
        document.getElementById('form-title').innerText = "내용 수정 중...";
        calcTotal();
        window.scrollTo(0, 0); // 상단 입력창으로 이동
    }

    function loadList() {
        const list = JSON.parse(localStorage.getItem('memberList')) || [];
        const tbody = document.getElementById('listBody');
        tbody.innerHTML = "";

        list.forEach((m, index) => {
            const v25 = parseInt(m.amt2025) || 0;
            const v26 = parseInt(m.amt2026) || 0;
            const total = v25 + v26;
           
            tbody.innerHTML += `<tr>
                <td class="col-name">${m.name}</td>
                <td class="col-pos">${m.position}</td>
                <td class="col-loc">${m.loc1}</td>
                <td class="col-amt">${v25.toLocaleString()}</td>
                <td class="col-amt">${v26.toLocaleString()}</td>
                <td class="col-total"><b>${total.toLocaleString()}원</b></td>
                <td class="col-action">
                    <button class="edit-btn" onclick="editData(${index})">수정</button>
                    <button class="del-btn" onclick="deleteData(${index})">삭제</button>
                </td>
            </tr>`;
        });
    }

    function filterList() {
        const query = document.getElementById('search').value.toLowerCase();
        const rows = document.getElementById('listBody').getElementsByTagName('tr');
        for (let row of rows) {
            const name = row.cells[0].innerText.toLowerCase();
            row.style.display = name.includes(query) ? "" : "none";
        }
    }

    function resetForm() {
        document.getElementById('edit-index').value = "-1";
        document.getElementById('save-btn').innerText = "저장하기";
        document.getElementById('form-title').innerText = "회비납부현황";
        document.querySelectorAll('input').forEach(input => {
            if(input.type !== 'hidden') input.value = "";
        });
        document.getElementById('total-amount').innerText = "0원";
    }

    window.onload = loadList;
</script>

</body>
</html>
