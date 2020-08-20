### MP4 文件格式

#### 关键概念

MP4 文件格式中，所有的内容存在一个称为movie 的容器中。一个movie 可以由多个tracks 组成。每个track 就是一个随时间变化的媒体序列，例如，视频帧序列。track 里的每个时间单位是一个sample，它可以是一帧视频，或者音频。sample 按照时间顺序排列。注意，一帧音频可以分解成多个音频sample，所以音频一般用sample 作为单位，而不用帧。MP4 文件格式的定义里面，用sample 这个单词表示一个时间帧或者数据单元。每个track 会有一个或者多个sample descriptions。track 里面的每个sample 通过引用关联到一个sample description。这个sample descriptions 定义了怎样解码这个sample，例如使用的压缩算法。与其他的多媒体文件格式不同的是，MP4 文件格式经常使用几个不同的概念，理解其不同是理解这个文件格式的关键。

 这个文件的物理格式没有限定媒体本身的格式。例如，许多文件格式将媒体数据分成帧，头部或者其他数据紧紧跟随每一帧视频，！！！TODO（例如MPEG2）。而MP4 文件格式不是如此。文件的物理格式和媒体数据的排列都不受媒体的时间顺序的限制。视频帧不需要在文件按时间顺序排列。这就意味着如果文件中真的存在这样的一些帧，那么就有一些文件结构来描述媒体的排列和对应的时间信息。

MP4 文件中所有的数据都封装在一些box 中（以前叫atom）。所有的metadata(媒体描述元数据)，包括定义媒体的排列和时间信息的数据都包含在这样的一些结构box 中。MP4 文件格式定义了这些这些box 的格式。Metadata 对媒体数据（例如，视频帧）引用说明。媒体数据可以包含在同一个的一个或多个box 里，也可以在其他文件中，metadata 允许使用URLs 来引用其他的文件，而媒体数据在这些引用文件中的排列关系全部在第一个主文件中的metadata 描述。其他的文件不一定是MP4 文件格式，例如，可能就没有一个box。

有很多种类的track，其中有三个最重要，video track 包含了视频sample；audio track包含了audio sample；hint track 稍有不同，它描述了一个流媒体服务器如何把文件中的媒体数据组成符合流媒体协议的数据包。如果文件只是本地播放，可以忽略hint track，他们只与流媒体有关系。

#### 媒体文件的物理结构

Box 定义了如何在sample table 中找到媒体数据的排列。这包括data reference(数据引用), the sample size table, the sample to chunk table, and the chunkoffset table. 这些表就可以找到track 中每个sample在文件中的位置和大小。

data reference 允许在第二个媒体文件中找到媒体的位置。这样，一部电影就可以由一个媒体数据库中的多个不同文件组成，而且不用把它们全部拷贝到另一个新文件中。例如，对视频编辑就很有帮助。为了节约空间，这些表都很紧凑。另外，interleave 不是sample by sample，而是把单个track 的几个samples 组合到一起，然后另外几个sample 又进行新的组合，等等。一个track 的连续几个sample 组成的单元就被称为chunk。每个chunk 在文件中有一个偏移量，这个偏移量是从文件开头算起的，在这个chunk 内，sample 是连续存储的。这样，如果一个chunk 包含两个sample，第二个sample 的位置就是chunk 的偏移量加上第一个sample 的大小。chunk offset table 说明了每个chunk 的偏移量，sample to chunk table说明了sample 序号和chunk 序号的映射关系。

注意chunk 之间可能会有死区，没有任何媒体数据引用到这部分区域，但是chunk 内部不会有这样的死区。这样，如果在节目编辑的时候，不需要一些媒体数据，就可以简单的留在那里，而不用引用，这样就不用删除它们了。类似的，如果媒体存放在第二个文件中，但是格式不同于MP4 文件格式，这个陌生文件的头部或者其他文件格式都可以简单忽略掉。

#### Temporal structure of the media

文件中的时间可以理解为一些结构。电影以及每个track 都有一个timescale。它定义了一个时间轴来说明每秒钟有多少个ticks。合理的选择这个数目，就可以实现准确的计时。一般来说，对于audiotrack，就是audio 的sampling rate。对于video track，情况稍微复杂，需要合理选择。例如，如果一个media TimeScale是30000，media sample durations 是1001，就准确的定义了NTSC video 的时间格式（虽然不准确，但一般就是29.97），and provide 19.9 hours of time in 32bits.

Track 的时间结构受一个edit list 影响，有两个用途：全部电影中的一个track 的一部分时间片断变化（有可能是重用）；空白时间的插入，也就是空的edits。特别注意的是如果一个track不是从节目开头部分开始，edit list 的第一个edit 就一定是空的edit。每个track的全部duration 定义在文件头部，这就是对track 的总结，每个sample 有一个规定的duration。一个sample 的准确描述时间，也就是他的时间戳(time-stamp)就是以前的sample 的duration 之和。

#### Interleave

 文件的时间和物理结构可以是对齐的，这表明媒体数据在容器中的物理顺序就是时间顺序。另外，如果多个track 的媒体数据包含在同一个文件中，这个媒体数据可以是interleaved。一般来说，为了方便读取一个track 的媒体数据，同时保证每个表紧凑，以一个合适的时间间隔（例如1 秒）做一次interleave，而不是sample by sample。这样就可以减少chunk 的数据，减小chunk offset table 的大小。

#### Composition

 如果多个audio track 包含在同一个文件中，他们有可能被混合在一起进行播放，并且由一个总track volume 和左/右balance控制。

类似的，video track 也可以根据各自的层次序列号（从后向前）和合成模式进行混合。另外，每个track 可以用一个matrix 进行变换，也可以全部电影用一个matrix 进行变换。这样既可以进行简单操作（例如放大图像，校正90º旋转），也可以做更复杂的操作（例如shearing,arbitrary rotation）。这个混合方法只是非常简单，是一个缺省的方法，MPEG4 的另一份文档会定义更强有力的方法（例如MPEG-4 BIFS）。

原文链接：https://blog.csdn.net/simongyley/java/article/details/8508267
