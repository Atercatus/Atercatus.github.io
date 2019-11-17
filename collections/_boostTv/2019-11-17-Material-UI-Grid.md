---
title: " Material UI Grid 전략"
excerpt: " Material UI Grid 전략"
last_modified_at: 2019-11-17T19:34:00
---

## Material-UI Grid 전략

Desktop 에서 좌우의 여백을 차지하는 Grid를 구성하기 위해 Grid item 내에 Grid Container를 배치합니다.
각각의 내부 Grid Container는 Desktop 기준으로 하나의 Row 를 나타냅니다.
이 때 Mobile 환경에서는 2줄(4개) 의 item이 하나의 Container에 둘러싸져있으므로 각각의 Tag들의 간격을 맞추기 귀한 추가적인 css 작업이 필요합니다. 따라서, 기존에 item에 적용되는 space에 맞춰서 Grid container에 margin을 줍니다.

```tsx
<Layout>
  <S.TagList>
    <S.ContainerGrid container spacing={0} justify="center">
      <Grid item xs={12} md={12}>
        <S.ContainerGrid container spacing={2} justify={"center"}>
          <Grid item xs={6} md={2}>
            <S.Tag>
              <S.TagTitle>python</S.TagTitle>
              <S.TagCount>2,292</S.TagCount>
            </S.Tag>
          </Grid>
          <Grid item xs={6} md={2}>
            <S.Tag>TEST2 </S.Tag>
          </Grid>
          <Grid item xs={6} md={2}>
            <S.Tag>TEST3 </S.Tag>
          </Grid>
          <Grid item xs={6} md={2}>
            <S.Tag>TEST4 </S.Tag>
          </Grid>
        </S.ContainerGrid>

        <S.ContainerGrid container spacing={2} justify={"center"}>
          <Grid item xs={6} md={2}>
            <S.Tag>TEST5 </S.Tag>
          </Grid>
          <Grid item xs={6} md={2}>
            <S.Tag>TEST6 </S.Tag>
          </Grid>
          <Grid item xs={6} md={2}>
            <S.Tag>TEST7 </S.Tag>
          </Grid>
          <Grid item xs={6} md={2}>
            <S.Tag>TEST8 </S.Tag>
          </Grid>
        </S.ContainerGrid>

        <S.ContainerGrid container spacing={2} justify={"center"}>
          <Grid item xs={6} md={2}>
            <S.Tag>TEST </S.Tag>
          </Grid>
          <Grid item xs={6} md={2}>
            <S.Tag>TEST </S.Tag>
          </Grid>
          <Grid item xs={6} md={2}>
            <S.Tag>TEST </S.Tag>
          </Grid>
          <Grid item xs={6} md={2}>
            <S.Tag>TEST </S.Tag>
          </Grid>
        </S.ContainerGrid>
      </Grid>
    </S.ContainerGrid>
  </S.TagList>
</Layout>
```
