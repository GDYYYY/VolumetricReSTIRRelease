+ capture时可以获取信息存为exr文件
+ exr文件的读取方式见./DATA
但没找到在哪存的

+ 从framecapture.cpp中发现有getRenderContext函数，疑似

+ 在captureAllOutputs发现
```c++
for (uint32_t i = 0 ; i < pGraph->getOutputCount() ; i++)
        {
            Texture* pTex = pGraph->getOutput(i)->asTexture().get();
            assert(pTex);
            auto ext = Bitmap::getFileExtFromResourceFormat(pTex->getFormat());
            auto format = Bitmap::getFormatFromFileExtension(ext);
            std::string filename = getOutputNamePrefix(pGraph->getOutputName(i)) + std::to_string(gpFramework->getGlobalClock().getFrame()) + "." + ext;
            pTex->captureToFile(0, 0, filename, format);
        }
```
`getOutputNamePrefix(pGraph->getOutputName(i))`应该是取决于py中的edge

验证VolumetricReSTIR.cpp中确有
```c++
const Falcor::ChannelList kOutputChannels =
    {
        { kAccumulatedColorOutput,     "gOutputFrame",    "accumulated output color (linear)", true /* optional */      },
        { kMotionVec,     "gMotionVec",    "motion vector", true /* optional */, ResourceFormat::RG32Float      }
    };
```
对应了已有输出accumulated_color和mvec


在cpp文件中，`vars["CB"]["gResolution"]=..`等为slang中输入赋值，然后通过`mTemporalReusePass->execute(pRenderContext, (int)scrWidth , (int)scrHeight );`调用，中间结果存在RenderContext中

直接这样传入引用然后slang里面修改即可
vars["gOutputDepth1"] = renderData[kDepth]->asTexture();

但是没有存下来图
原因是使用的2022进行build并未成功更新mogwai.exe

每次修改后使用2019清除并重新生成解决方案

从depth的截图来看，引擎里面的顺序应该是BGR