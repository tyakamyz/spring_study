> ## UriComponentsBuilder를 이용하는 링크 생성
- 여러개의 파라미터들을 연결해서 uri의 형태로 만들어 주는 기능
    > Criteria.java
    ```java
    package org.zerock.domain;

    import org.springframework.web.util.UriComponentsBuilder;

    import lombok.Getter;
    import lombok.Setter;
    import lombok.ToString;

    @Getter
    @Setter
    @ToString
    public class Criteria {
        private int pageNum;
        private int amount;
        
        private String type;
        private String keyword;
        
        public Criteria() {
            this(1,10);
        }
        
        public Criteria(int pageNum, int amount) {
            this.pageNum = pageNum;
            this.amount = amount;
        }
        
        public String[] getTypeArr() {
            return type == null ? new String[] {} : type.split("");
        }
        
        public String getListLink() {
            UriComponentsBuilder builder = UriComponentsBuilder.fromPath("")
                .queryParam("pageNum", this.pageNum)
                .queryParam("amount", this.amount)
                .queryParam("type", this.getType())
                .queryParam("keyword", this.getKeyword());
            return builder.toUriString();
        }
    }
    ```
    > BoardController.java
    ```java
    @PostMapping("/modify")
    public String modify(BoardVO board, @ModelAttribute("cri") Criteria cri, RedirectAttributes rttr) {
        log.info("modify: " + board);
        
        if(service.modify(board)) {
            rttr.addFlashAttribute("result", "success");
        }
        
        //rttr.addFlashAttribute("pageNum", cri.getPageNum());
        //rttr.addFlashAttribute("amount", cri.getAmount());
        //rttr.addFlashAttribute("type", cri.getType());
        //rttr.addFlashAttribute("keyword", cri.getKeyword());
        
        return "redirect:/board/list" + cri.getListLink();
    }

    @PostMapping("/remove")
    public String remove(@RequestParam("bno") Long bno, @ModelAttribute("cri") Criteria cri, RedirectAttributes rttr) {
        log.info("remove....." + bno);
        
        if(service.remove(bno)) {
            rttr.addFlashAttribute("result", "success");
        }
        
        //rttr.addFlashAttribute("pageNum", cri.getPageNum());
        //rttr.addFlashAttribute("amount", cri.getAmount());
        //rttr.addFlashAttribute("type", cri.getType());
        //rttr.addFlashAttribute("keyword", cri.getKeyword());
        
        return "redirect:/board/list" + cri.getListLink();
    }
    ```